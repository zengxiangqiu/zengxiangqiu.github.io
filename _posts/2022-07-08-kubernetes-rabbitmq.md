---
 title: "kubernetes rabbitmq"
 date:  2022-07-08 11:44:57 +0800
 categories: [k8s]
 tags: [rabbit]
---


## 部署

`kubectl apply -f https://github.com/rabbitmq/cluster-operator/releases/latest/download/cluster-operator.yml`

`kubectl apply -f https://raw.githubusercontent.com/rabbitmq/cluster-operator/main/docs/examples/hello-world/rabbitmq.yaml`

```yaml
apiVersion: rabbitmq.com/v1beta1
kind: RabbitmqCluster
metadata:
  name: lsd-rabbitmq
```

## 用户密码

查看default account 和 password

`username="$(kubectl get -n rabbitmq-system secret lsd-rabbitmq-default-user -o jsonpath='{.data.username}' | base64 --decode)"`

`password="$(kubectl get -n rabbitmq-system secret lsd-rabbitmq-default-user -o jsonpath='{.data.password}' | base64 --decode)"`

添加用户

`rabbitmqctl add_user default_user_TCxQBnkrxpLW73cy0Tl ELxsVWcFNZM-cxObUxPdNby8ImsSU9Ta`


## TCP 端口转发

1. tcp-services

    因为 Ingress does not support TCP or UDP services., ingress-nginx-controller 启动时引用 tcp-services, 修改 configmap

    ```yaml
    apiVersion: v1
    kind: ConfigMap
    metadata:
      name: tcp-services
      namespace: ingress-nginx
    data:
      5672: "rebbitmq-system/lsd-rabbitmq:5672"
      15672: "rebbitmq-system/lsd-rabbitmq:15672"
    ```



    或者打补丁

    `kubectl patch configmap tcp-services -n ingress-nginx --patch '{"data":{"15672": "rabbitmq-system/lsd-rabbitmq:15672"}}'`

    重启 Deployment ingress-nginx-controller

2. port-forward

    `kubectl port-forward pod/lsd-rabbitmq --address=172.21.xx.xx  15672:15672`

## 清除污点

`kubectl patch crd/rabbitmqclusters.rabbitmq.com -p '{"metadata":{"finalizers":[]}}' --type=merge`






## 步骤

`kubectl apply -f https://github.com/rabbitmq/cluster-operator/releases/latest/download/cluster-operator.yml`
`kubectl apply -f https://raw.githubusercontent.com/rabbitmq/cluster-operator/main/docs/examples/hello-world/rabbitmq.yaml`

```bash
[root@master ~]# kubectl describe pod/lsd-rabbitmq-server-01 -n rabbitmq-system
[root@master ~]# kubectl get pvc
[root@master ~]# kubectl get pv
```

发现Container无法绑定PVC rabbitmq-storage-rabbitmq-cluster-0，也没有申领到pv，不存在 rabbitmq-storage-rabbitmq-cluster-0

查看[Troubleshooting Cluster Operator](https://www.rabbitmq.com/kubernetes/operator/troubleshooting-operator.html)，判断应该是因为`Incorrect storageClassName configuration`引起

查看[persistence](https://www.rabbitmq.com/kubernetes/operator/using-operator.html#persistence)，尝试Apply配置，关于 kubectl Create vs apply 的区别查看[kubectl apply vs kubectl create](https://stackoverflow.com/questions/47369351/kubectl-apply-vs-kubectl-create)，create 指令式管理，apply声明式管理

```yaml
apiVersion: rabbitmq.com/v1beta1
kind: RabbitmqCluster
metadata:
  name: lsd-rabbitmq
spec:
   persistence:
    storageClassName: fast
    storage: 20Gi
```
k8s default 没有任何StorageClassName， 要手动创建

查看 [nfs](https://kubernetes.io/zh-cn/docs/concepts/storage/storage-classes/#nfs),先创建存储制备器provisioner，注意`Kubernetes 不包含内部 NFS 驱动。你需要使用外部驱动为 NFS 创建 StorageClass`,

参考 [kubernetes-retired/external-storage](https://github.com/kubernetes-retired/external-storage/blob/master/nfs/deploy/kubernetes/deployment.yaml)，先apply rbac.yaml,创建clusterrole，role，rolebinding，再apply depolyment，创建 serviceaccount，deployment，再创建StorageClass， PersistentVolumeClaim

创建后发现 pvc 一直pending，查看logs，发现`waiting for a volume to be created, either by external provisioner "wangzy-nfs-storage" or manually created by system administrator` 或 `selfLink was empty, can't make reference`

参考 [selfLink was empty, can't make reference](https://github.com/kubernetes-sigs/nfs-subdir-external-provisioner/issues/25)，找到workaround的方法

edit /etc/kubernetes/manifests/kube-apiserver.yaml

```yaml
spec:
  containers:
  - command:
    - kube-apiserver
```
Add this line:
- --feature-gates=RemoveSelfLink=false

kubectl apply -f /etc/kubernetes/manifests/kube-apiserver.yaml

发现apiserver无法正常启动，查看日志，发现 k8s v1.20+ 禁用了selflink,所以此路不通

"selfLink is a URL representing this object. Populated by the system. Read-only. DEPRECATED Kubernetes will stop propagating this field in 1.20 release and the field is planned to be removed in 1.21 release."

有位网友提到

> Just use new available docker image: gcr.io/k8s-staging-sig-storage/nfs-subdir-external-provisioner:v4.0.0 and it works fine. Do not edit kube-apiserver.yaml, there is no need to.

尝试升级镜像库，因为gcr.io访问不到，改为 depolyment.yaml `docker.io/dyrnq/nfs-subdir-external-provisioner:v4.0.2`

创建StorageClass
```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: nfs-storage
  annotations:
    storageclass.kubernetes.io/is-default-class: "true"  #---设置为默认的storageclass
provisioner: fuseim.pri/ifs  #---动态卷分配者名称，必须和上面创建的"PROVISIONER_NAME"变量中设置的Name一致
parameters:
  archiveOnDelete: "true"  #---设置为"false"时删除PVC不会保留数据,"true"则保留数据
mountOptions:
  - hard        #指定为硬挂载方式
  - nfsvers=4   #指定NFS版本，这个需要根据 NFS Server 版本号设置
```

先 delete rabbitmq.yaml，再apply

```yaml
apiVersion: rabbitmq.com/v1beta1
kind: RabbitmqCluster
metadata:
  name: lsd-rabbitmq
  namespace: rabbitmq-system
spec:
  persistence:
    storageClassName: nfs-storage
    storage: 10Gi
```
查看pvc，已正常Bound

```bash
[root@master ~]# kubectl  get pvc
NAME                                  STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS          AGE
rabbitmq-storage-rabbitmq-cluster-0   Bound    pvc-03d69ccf-3ec3-4626-b131-f9617618ccc8   2Gi        RWX            managed-nfs-storage   23h
```

再查看pod status ，发现 pod container 处于Exist状态，查看pod 日志，发现`lsd-rabbitmq-server-0.lsd-rabbitmq-nodes.rabbitmq-system timeout`,参考[epmd error for host when starting rabbitmq](https://www.programmerall.com/article/7871569089/),要将这个域名添加入/etc/hosts

```bash
127.0.0.1     localhost, lsd-rabbitmq-server-0.lsd-rabbitmq-nodes.rabbitmq-system
```

要想办法设置host

参考 [Failed to get nodes from k8s](https://github.com/helm/charts/issues/17508)

```yaml
      hostAliases:
      - ip: 10.43.0.1
        hostnames:
        - kubernetes.default.svc.cluster.local
```

既然 persistence 可以在 rabbitmq.yaml 设置，hostAliases应该也可以，查看 cluster-operator.yml,搜索 hostAliases

结构如下：

```yaml
override:
  statefulSet:
    spec:
     template:
       spec:
         hostAliases:
         - ip: 127.0.0.1
           hostnames:
           - lsd-rabbitmq-server-0.lsd-rabbitmq-nodes.rabbitmq-system
```
修改rabbitmq.yaml

```yaml
apiVersion: rabbitmq.com/v1beta1
kind: RabbitmqCluster
metadata:
  name: lsd-rabbitmq
  namespace: rabbitmq-system
spec:
  override:
    statefulSet:
      spec:
        template:
          containers:
          - command:
              - /manager
            env:
              - name: OPERATOR_NAMESPACE
                valueFrom:
                  fieldRef:
                    fieldPath: metadata.namespace
            image: rabbitmqoperator/cluster-operator:2.0.0
            name: operator
            ports:
              - containerPort: 9782
                name: metrics
                protocol: TCP
            resources:
              limits:
                cpu: 200m
                memory: 500Mi
              requests:
                cpu: 200m
                memory: 500Mi
          spec:
            hostAliases:
            - ip: 127.0.0.1
              hostnames:
              - lsd-rabbitmq-server-0.lsd-rabbitmq-nodes.rabbitmq-system
  persistence:
    storageClassName: nfs-storage
    storage: 10Gi

```
containers 是必需的，复制cluster-operator.yml 中的deployment containers

再试，发现报 443 错误，应该是访问不到 apiserver

```bash
2019-09-28 11:07:57.715 [info] <0.224.0> Failed to get nodes from k8s - {failed_connect,[{to_address,{"kubernetes.default",443}},{inet,[inet],closed}]}
```

```bash
[root@master ~]# crictl ps
CONTAINER           IMAGE               CREATED             STATE               NAME                      ATTEMPT             POD ID              POD
edec02162b08a       5185b96f0becf       6 days ago          Running             coredns                   0                   3995f75c191f5       coredns-c676cc86f-p7vdl
6fa312b3b8aa1       5185b96f0becf       6 days ago          Running             coredns                   0                   3b8c9b069e1de       coredns-c676cc86f-g4d49
[root@master ~]# crictl inspect edec02162b08a
evn:[
  ....
  "KUBERNETES_PORT_443_TCP=tcp://10.96.0.1:443"
]
```

接下来修改rabbitmq.yaml

```yaml
            hostAliases:
            - ip: 127.0.0.1
              hostnames:
              - lsd-rabbitmq-server-0.lsd-rabbitmq-nodes.rabbitmq-system
            - ip: 10.96.0.1
              hostnames:
              - kubernetes.default
```

成功run pod

尝试apply service ,将 rabbitmq UI 公布到外网

```yaml
kind: Service
apiVersion: v1
metadata:
  labels:
    app.kubernetes.io/name: lsd-rabbitmq #这里要注意 key:value ,我们查看describe是 key=value
  name: rabbitmq-cluster-manage
  namespace: rabbitmq-system #namespace
spec:
  ports:
  - name: http
    port: 15672
    protocol: TCP
    targetPort: 15672
  selector:
    app.kubernetes.io/name: lsd-rabbitmq  #这里要注意 key:value ,我们查看describe是 key=value
  type: NodePort
```

```bash
[root@master ~]# kubectl  get services -n rabbitmq-system
NAME                      TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)                        AGE
lsd-rabbitmq              ClusterIP   10.104.215.205   <none>        5672/TCP,15672/TCP,15692/TCP   3h54m
lsd-rabbitmq-nodes        ClusterIP   None             <none>        4369/TCP,25672/TCP             3h54m
rabbitmq-cluster-manage   NodePort    10.99.41.142     <none>        15672:30907/TCP                3h10m
```

因为部署在 node1 ，所以访问 http://172.21.14.208:30907/#/

成功，查看user ，pwd ，登陆

`username="$(kubectl get -n rabbitmq-system secret lsd-rabbitmq-default-user -o jsonpath='{.data.username}' | base64 --decode)"`
`password="$(kubectl get -n rabbitmq-system secret lsd-rabbitmq-default-user -o jsonpath='{.data.password}' | base64 --decode)"`





## 参考

[RabbitMQ Cluster Kubernetes Operator](https://github.com/rabbitmq/cluster-operator)

[端口转发](https://minikube.sigs.k8s.io/docs/tutorials/nginx_tcp_udp_ingress/)

[RabbitMQ Cluster Operator for Kubernetes](https://www.rabbitmq.com/kubernetes/operator/operator-overview.html)
