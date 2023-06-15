---
title: "kubernetes install"
date: 2022-11-07 11:59:04 +0800
categories: [kubernetes]
tags: [kubernetes]
---

## kubectl

常见命令

```bash
#集群健康
kubectl get cs
#创建deployment 以及service
kubectl create deployment demo --image=httpd --port=80
kubectl expose deploy  [deployName]  --name=xxx --type=NodePort
#services Endpoints
kubectl get svc,ep
#重启depolyment
kubectl rollout restart deployment nginx-deployment
#标签，日志输出
kubectl logs -l app=lsd-inventory -n lesaunda
#以机器可读的方式列举隶属于某 Job 的全部 Pod
pods=$(kubectl get pods --selector=job-name=pi --output=jsonpath='{.items[*].metadata.name}')
echo $pods
kubectl logs $pods
# scale deployment
kubectl scale deployment/xxxx --replicas=5 -n lesaunda
# find out default namespace
kubectl config view --minify --output 'jsonpath={..namespace}'; echo
# set default namespace
kubectl config set-context --current --namespace=<NAME>
```

1. localhost:8080 was refused

   ```bash
   docker ps | grep kube-apiserver

   mkdir -p $HOME/.kube
   sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
   sudo chown $(id -u):$(id -g) $HOME/.kube/config
   ```

   参考[The connection to the server localhost:8080 was refused - did you specify the right host or port?](https://discuss.kubernetes.io/t/the-connection-to-the-server-localhost-8080-was-refused-did-you-specify-the-right-host-or-port/1464)

## kubeadm init

```
yum install kubelet-1.25.3  kubeadm-1.25.3  kubectl-1.25.3 --nogpgcheck -y
```

为了保证 kubelet 正常工作，你必须禁用交换分区,关闭 selinux 与防火墙,配置 kubernetes /etc/modules-load.d/k8s.conf

`kubeadm init --kubernetes-version=v1.25.3 --image-repository=registry.aliyuncs.com/google_containers --pod-network-cidr=10.244.0.0/16 --v=9`

1. 镜像问题

   ```bash
   kubeadm config images list
   kubeadm config images pull --cri-socket=unix:///run/containerd/containerd.sock --image-repository=registry.aliyuncs.com/google_containers --kubernetes-version=v1.25.3
   ```

2. Init 失败

   `systemctl status kubelet` 状态 Actived，正常，如 InActivce, `systemctl enable kubelet`

   `journalctl -xeu kubectl --no-pager` 查看日志，发现`failed to pull image [registry.k8s.io/pause:3.6 -o pause.tar]`

   拉取，tag，import

   ```bash
   crictl images
   docker pull registry.aliyuncs.com/google_containers/pause:3.6
   docker tag registry.aliyuncs.com/google_containers/pause:3.6 registry.k8s.io/pause:3.6
   docker save registry.k8s.io/pause:3.6 -o pause.tar
   ctr -n k8s.io  images import pause.tar
   ctr -n k8s.io i tag registry.aliyuncs.com/google_containers/pause:3.6 registry.k8s.io/pause:3.6

   ```

3. failed to reserve sandbox name

   apiserver 访问 container runtime interface 失败

   `vim /etc/containerd/config.toml`

   注释

   #disabled_plugins = ["cri"]

   新增

   level = "debug"

   `systemctl daemon-reload`

   `systemctl  restart containerd`

4. 节点 Role

   `kubectl get no master -o wide --show-labels`

   `kubectl get nodes`

   参考[k8s 中节点 node 的 ROLES 值是 none](https://blog.csdn.net/Lingoesforstudy/article/details/116484624)

5. /proc/sys/net/bridge/bridge-nf-call-iptables contents are not set to 1

   执行`echo "1" >/proc/sys/net/bridge/bridge-nf-call-iptables`

## CoreDNS

```bash
kubectl run -it --image=busybox:1.28.3 --rm --restart=Never bash -n kube-system
nslookup  kubernetes.default

:: no server could be reached.

kubectl get ep kube-dns -n kube-system -o json |jq -r ".subsets"
```

coredns 服务/pod 正常，但创建的 pod 无法 nslook 服务，coredns configmap 加 log 也检测不到，反而 master node 上`dig @10.96.0.10 kubernetes.default.svc.cluster.local +noall +answer` 可以找到域名的 ip,无果，删除 dns pod 后自动创建，恢复正常。怀疑与期间修改 flannel subnet 文件以及 cni0 网络相关,尝试 改 ipv4 允许转发,busybox image 应<=1.28.3, 大于此版本可能出现 No Answer 问题

2023-05-23 补充，node2 存在多个 brige network, ping 192.168.20.xx 时 出现

```bash
ping 192.168.20.xx

From 192.168.16.1 icmp seq =1  destination host unreachable
From 192.168.16.1 icmp seq =1  destination host unreachable
```

```bash
# 查看iptables 规则,没有发现异常
iptables -L -n

# firewall status inactive
systemctl status firewalld.service

# 安装bind_utils工具，包括nslookup, dig
yum install bind_utils

# dig <ip>, nslookup ip
1 node2 3000ms !H

# ip a 或者 ifconfig 或 安装  bridge-utils, 看到brige network 关联 192.168.16.1
brctl show

# delete
ip link set <bridge_name> down
ip link del <bridge_name>

# 或者nmcli
nmcli con show
nmcli connecion down <bridge_name>

# ping ok

```

## crictl

参考[使用 crictl 对 Kubernetes 节点进行调试](https://kubernetes.io/zh-cn/docs/tasks/debug/debug-cluster/crictl/)

Install

```sh
VERSION="v1.26.0"
wget https://github.com/kubernetes-sigs/cri-tools/releases/download/$VERSION/crictl-$VERSION-linux-amd64.tar.gz
sudo tar zxvf crictl-$VERSION-linux-amd64.tar.gz -C /usr/local/bin
rm -f crictl-$VERSION-linux-amd64.tar.gz
```

```bash
crictl ps
crictl images
crictl inspect [container_id]
crictl inspecti [image_id]
crictl inspectp [pod_id]
ctr --namespace=k8s.io image tag  registry.aliyuncs.com/google_containers/pause:3.6 registry.k8s.io/pause:3.6
```

`vim /etc/crictl.yaml`

```json
runtime-endpoint: unix:///run/containerd/containerd.sock
image-endpoint: unix:///run/containerd/containerd.sock
timeout: 0
debug: true #调试
pull-image-on-create: false
disable-pull-on-run: false

```
```bash
$ crictl ps

I0605 14:56:16.720887  254196 util_unix.go:103] "Using this endpoint is deprecated, please consider using full URL format" endpoint="/run/containerd/containerd.sock" URL="unix:///run/containerd/containerd.sock"

$ crictl config --set runtime-endpoint=unix:///run/containerd/containerd.sock

```


## 组件 etcd

由 kubeadm 生成证书，kubelet + cri 管理 etcd 服务，手动创建清单

`kubeadm init phase etcd local --config=/tmp/${HOST0}/kubeadmcfg.yaml`

install kubeadm 时 生成 /etc/kubernetes/manifests/etcd.yaml，可以参考

利用 etcdctl 工具查看集群成员

```sh


export ENDPOINTS="192.168.5.41:2379,192.168.5.45:2379,192.168.5.46:2379"
etcdctl --write-out=table --endpoints=$ENDPOINTS member list


ETCDCTL_API=3 etcdctl --cert /etc/kubernetes/pki/etcd/peer.crt --key /etc/kubernetes/pki/etcd/peer.key --cacert /etc/kubernetes/pki/etcd/ca.crt --endpoints https://127.0.0.1:2379 member list



```

join master 过程中出现 etcd 出现 no leader 问题 和 tls connecion refused ,与证书有关，不要轻易改动 /etc/kubernetes/pki 目录，改的话要先备份。

没有太好的处理方法

--force-new-cluster 'false'
Force to create a new one-member cluster.

利用备份文件和 etcd.yaml 在 kubernetes 外重新搭建 etcd 实例

```sh
/tmp/etcd-download-test/etcd  --listen-client-urls=http://172.21.14.73:2379   --advertise-client-urls=http://172.21.14.73:2379  --client-cert-auth=false --peer-client-cert-auth=false  --name=master --initial-cluster=master=http://172.21.14.73:2380 --initial-advertise-peer-urls=http://172.21.14.73:2380  --data-dir=/var/lib/etcd_bak --initial-cluster-state existing --force-new-cluster='true'
```

不验证 auth，apiserver 的 yaml 文件中 etc listen url 也相应改为 http://127.0.0.1:2379 ，测试通过

[手动建 etcd 集群](https://kubernetes.io/zh-cn/docs/setup/production-environment/tools/kubeadm/setup-ha-etcd-with-kubeadm/)

[Configuration options](https://etcd.io/docs/v3.5/op-guide/configuration/)

[How to Add and Remove Members](https://etcd.io/docs/v3.5/tutorials/how-to-deal-with-membership/)

[如何安装 etcd 和 etcdctl](https://github.com/etcd-io/etcd/releases/)

## kubeadm join

```bash
# 生成加入集群命令
kubeadm token create --print-join-command

# 加入集群 (过时)
kubeadm join 172.21.14.73:6443 --token i7guj6.at1q41k4bdgi0gbg --discovery-token-ca-cert-hash sha256:fc2a4732767762ea293ff2c6da67162bcc27e9850fc4919e769b0215c300d856 --cri-socket unix:///run/containerd/containerd.sock --v=8

# 将控制平面证书上传到集群。默认情况下，证书和加密密钥会在两个小时后过期。
kubeadm init phase upload-certs --upload-certs --v=5

# 将上述命令返回的key 作为--certificate-key 参数 ，--control-plane 加入控制面，此节点不部署pod
kubeadm join 172.21.14.73:6443 --token 0q5r70.53fggw68nta4697x --discovery-token-ca-cert-hash sha256:fc2a4732767762ea293ff2c6da67162bcc27e9850fc4919e769b0215c300d856 --certificate-key 27f45c69543808830fb2c59c2dd110a7e010766dd1e5a1ea9a956ef58a194bd0 --control-plane

# 签署证书
[certs] Using certificateDir folder "/etc/kubernetes/pki"
[certs] Generating "etcd/healthcheck-client" certificate and key
[certs] Generating "etcd/server" certificate and key
[certs] etcd/server serving cert is signed for DNS names [gz-k8ssrv-master2 localhost] and IPs [192.168.1.192 127.0.0.1 ::1]
[certs] Generating "etcd/peer" certificate and key
[certs] etcd/peer serving cert is signed for DNS names [gz-k8ssrv-master2 localhost] and IPs [192.168.1.192 127.0.0.1 ::1]
[certs] Generating "apiserver-etcd-client" certificate and key
[certs] Generating "apiserver" certificate and key
[certs] apiserver serving cert is signed for DNS names [gz-k8ssrv-master2 kubernetes kubernetes.default kubernetes.default.svc kubernetes.default.svc.cluster.local] and IPs [10.96.0.1 192.168.1.192 172.21.14.73]
[certs] Generating "apiserver-kubelet-client" certificate and key
[certs] Generating "front-proxy-client" certificate and key
[certs] Valid certificates and keys now exist in "/etc/kubernetes/pki"
[certs] Using the existing "sa" key
```

1. Port 10250 is in use

   `kubeadm reset` 再 join

## containerd

```bash

yum-config-manager --add-repo http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo

sudo yum install docker-ce docker-ce-cli containerd.io docker-compose-plugin

sudo yum install containerd

systemctl enable containerd

systemctl start containerd
```

```bash
vim /etc/containerd/config.toml
注释,开放接口给kubelet创建管理container
#disabled_plugins = [“cri”]
或者
sudo sed -i 's/^disabled_plugins \=/\#disabled_plugins \=/g' /etc/containerd/config.toml

sandbox_image = "registry.cn-hangzhou.aliyuncs.com/google_containers/pause:3.8"   # 修改为阿里云镜像地址
SystemdCgroup = true                                                              # 使用systemd cgroup

镜像
[plugins]
  [plugins.cri]
    [plugins.cri.registry]
        [plugins.cri.registry.mirrors]
          [plugins.cri.registry.mirrors."registry.k8s.io"]
            endpoint = ["https://registry.cn-hangzhou.aliyuncs.com/google_containers"]
          [plugins.cri.registry.mirrors."docker.io"]
            endpoint = ["https://registry.cn-hangzhou.aliyuncs.com"]

```

`systemctl daemon-reload`
`systemctl enable --now containerd` 守护进程
`systemctl restart containerd`

## docker

```bash
vim /etc/docker/daemon.json

{
  "exec-opts": ["native.cgroupdriver=systemd"],
  "insecure-registries": ["172.21.14.206:5000"],
  "registry-mirrors": ["https://hub-mirror.c.163.com"]
}

systemctl daemon-reload
systemctl restart docker
```

## Dashboard

安装

`kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/v2.7.0/aio/deploy/recommended.yaml`

修改 service 成 nodeport，外网访问

```bash
kubectl describe pod/kubernetes-dashboard

#Normal   Scheduled         42m                  default-scheduler  Successfully assigned kubernetes-dashboard/>
#kubernetes-dashboard-5c8bd6b59-xxvkk to node1

kubectl cluster-info

Kubernetes control plane is running at https://172.21.14.73:6443
CoreDNS is running at https://172.21.14.73:6443/api/v1/namespaces/kube-system/services/kube-dns:dns/proxy

kubectl proxy

# node1-> 172.21.14.208
https://172.21.14.208:30423/#/login
```

参考[Accessing Dashboard](https://github.com/kubernetes/dashboard/blob/master/docs/user/accessing-dashboard/README.md)

1. nodeport ,token 登陆

   按[k8s 优秀的 web 管理界面-kubernetes Dashboard](https://cloud.tencent.com/developer/article/1919416) apply

   `kubectl get secret -n kubernetes-dashboard` 没有找到 token

   1.23 以上执行

   创建 serviceaccount

   ```yaml
   apiVersion: v1
   kind: ServiceAccount
   metadata:
     name: admin
     namespace: kubernetes-dashboard

   ---
   apiVersion: rbac.authorization.k8s.io/v1
   kind: ClusterRoleBinding
   metadata:
     name: admin
   roleRef:
     apiGroup: rbac.authorization.k8s.io
     kind: ClusterRole
     name: cluster-admin
   subjects:
     - kind: ServiceAccount
       name: admin
       namespace: kubernetes-dashboard
   ```

   `kubectl -n kubernetes-dashboard create token admin` 生成 token

## 网络插件 flannel

参考 [扁平网络 Flannel](https://jimmysong.io/kubernetes-handbook/concepts/flannel.html)

安装

```bash
curl -o kube-flannel.yml https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml
kubectl apply -f kube-flannel.yml
docker pull quay.io/coreos/flannel:v0.14.0
kubectl apply -f kube-flannel.yml
```

1. open /run/flannel/subnet.env no such file or directory

   新建 /run/flannel/subnet.env 文件

   ```json
   FLANNEL_NETWORK=10.244.0.0/16
   FLANNEL_SUBNET=10.244.0.1/24
   FLANNEL_MTU=1450
   FLANNEL_IPMASQ=true
   ```

   注意到 kube-flannel pod 启动后会重新覆盖此文件内容，不同的 node，FLANNEL_SUBNET 会有所不同`10.244.1.1/24,10.244.2.1/24`

2. k8s 安装 flannel 报错“node "master" pod cidr not assigned”

   Kubeadm Init 的时候，没有增加 --pod-network-cidr 10.244.0.0/16

3. apply pod 时出现 `failed to set bridge addr: "cni0" already has an IP address different from 10.244.1.1/24`

   修改 cni0 网络

   ```bash
   ifconfig cni0 down
   ifconfig flannel.1 down
   ip link delete cni0
   ip link delete flannel.1
   ifconfig cni0
   ifconfig flannel.1

   ip link add cni0 type bridge
   ip link set dev cni0 up
   ifconfig cni0 10.244.0.1
   ifconfig cni0 mtu 1450 up
   ```

## 污点 node

kubernetes 提示 1 node(s) had taints that the pod didn't tolerate

正常情况下 master 不参与工作负载

`kubectl taint nodes --all node-role.kubernetes.io/master-`

[kubernetes 提示 1 node(s) had taints that the pod didn't tolerate](https://www.cnblogs.com/ip99/p/14231203.html)

[污点](https://kubernetes.io/zh-cn/docs/concepts/scheduling-eviction/taint-and-toleration/)

## Tools

```bash
yum -y install net-tools
```

## ingrees-nginx

### 部署

`kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.5.1/deploy/static/provider/cloud/deploy.yaml`

### 配置

group, 捕获第n个组

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: sysinfo-ingress
  namespace: lesaunda
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /$2
spec:
  ingressClassName: nginx
  rules:
  - http:
      paths:
      - pathType: Prefix
        path: /sysinfo/api(/|$)(.*)
        backend:
          service:
            name: ls-sysinfo
            port:
              number: 3000
```
正则匹配，保留 `/api/v1/staffs`

```yaml
annotations:
  nginx.ingress.kubernetes.io/use-regex: 'true'
spec:
 ingressClassName: nginx
  rules:
  - http:
      paths:
        - path: /api/v\d+/(staffs|accounts)
          pathType: Prefix
          backend:
            service:
              name: lsd-staff-services
              port:
                number: 80

```

### 问题

1. 镜像问题

   导出文件

   `curl -o ingress-nginx-deploy-backup.yaml https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.5.1/deploy/static/provider/cloud/deploy.yaml`

   register.k8s.io 无法访问，阿里云镜像代替，pull 之后修改 deployment.yaml

   `docker pull registry.cn-hangzhou.aliyuncs.com/google_containers/kube-webhook-certgen:v20220916-gd32f8c343`
   `docker pull registry.cn-hangzhou.aliyuncs.com/google_containers/nginx-ingress-controller:v1.5.1`

   apply 即可

2. Internal error occurred: failed calling webhook "validate.nginx.ingress.kubernetes.io"

   ```bash
   $ kubectl delete -A ValidatingWebhookConfiguration ingress-nginx-admission
   validatingwebhookconfiguration.admissionregistration.k8s.io "ingress-nginx-admission" deleted
   ```

   参考[Kubernetes: nginx ingress controller - failed calling webhook](https://pet2cattle.com/2021/02/service-ingress-nginx-controller-admission-not-found)

   [在 Minikube 环境中使用 NGINX Ingress 控制器配置 Ingress](https://kubernetes.io/zh-cn/docs/tasks/access-application-cluster/ingress-minikube/)

3. tcp 转发

   ```bash
   $kubectl  -n ingress-nginx get cm

   tcp-services               1      191d

   $kubectl -n ingrees-nginx edit cm tcp-services

   data:
     "5672": rabbitmq-system/lsd-rabbitmq:5672
   ```

4. 413 request entity too large

  上传文件，nginx 转发，size 超过 limit

  查看`kubectl exec -n ingress-nginx  <ingress pod> cat nginx.conf|grep client_max_body_size`

  `client_max_body_size 1m` default 1m

  修改`kubectl  -n ingress-nginx edit cm ingress-nginx-controller`

  ```yaml
  data:
    proxy-body-size: 10m
  ```

  相关配置可以参考 [Kubernetes NGINX Ingress: 10 Useful Configuration Options](https://loft.sh/blog/kubernetes-nginx-ingress-10-useful-configuration-options)

## redis cluster

参考[k8s 中部署 redis 集群(三主三从)](https://www.cnblogs.com/LiuChang-blog/p/15898005.html)

获取 pods ip 地址
`kubectl get pods -l app=redis-cluster -n redis-cluster -o jsonpath='{range.items[*]}{.status.podIP}:6379 '`

redis-cli -a [password] --cluster create 10.244.2.89:6379 10.244.2.90:6379 10.244.0.20:6379 10.244.2.91:6379 10.244.0.21:6379 10.244.2.92:6379 --cluster-replicas 1

如果所有 pod 下线，需要重新建群

1. `kubectl delete -f redis-cluster.yml` 删除资源
2. `kubectl delete pvc/data-redis-cluster-0 -n redis-cluster` 逐一删除 pvc
3. `rm -rf /nfs_root/redis-cluster-data-redis-cluster-*/` 删除 nfs 空间
4. `kubectl apply -f redis-cluster.yml` 重新建立集群

或者

1. `kubectl delete -f redis-cluster.yml` 删除资源
2. `rm -f /ifs/kubernetes/redis-cluster-data-redis-cluster-0*/nodes.conf*` 逐一删除 nodes.conf
3. `kubectl apply -f redis-cluster.yml` 重新建立集群

### redisinsight

参考[Install RedisInsight on Kubernetes](https://docs.redis.com/latest/ri/installing/install-k8s/)

[LoadBalancer](https://kubernetes.io/zh-cn/docs/concepts/services-networking/service/#loadbalancer)

> 要实现 type: LoadBalancer 的服务，Kubernetes 通常首先进行与请求 type: NodePort 服务等效的更改。 cloud-controller-manager 组件然后配置外部负载均衡器以将流量转发到已分配的节点端口。

访问 http://172.21.14.208:30738/



###  Install as node

1. `mv /etc/yum.repos.d/CentOS-Base.repo /etc/yum.repos.d/CentOS-Base.repo.bak`
```bash
$ wget -O /etc/yum.repos.d/CentOS-Base.repo https://mirrors.aliyun.com/repo/Centos-7.repo
$ yum clean all
$ yum makecache
```
`yum update`
2. network，[安装和配置先决条件](https://kubernetes.io/zh-cn/docs/setup/production-environment/container-runtimes)
3. 关闭防火墙 `systemctl disable firewall.service`
4. setenforce 0 #关闭安全模式
5. swapoff -a #关闭内存交换
6. 配置docker.repo  `yum-config-manager --add-repo http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo`
7. install containerd(preinstall runc and cni plugin), runc centos 7 自带（runc --version 1.1.7）， cni plugin flannel 由 kubelet 调度部署, enable 以及start
8. 配置 `/etc/yum.repos.d/kubernetes.repo`
   ```ymal
    [kubernetes]
    name=Kubernetes
    baseurl=https://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86_64/
    enabled=1
    gpgcheck=1
    repo_gpgcheck=1
    gpgkey=https://mirrors.aliyun.com/kubernetes/yum/doc/yum-key.gpg https://mirrors.aliyun.com/kubernetes/yum/doc/rpm-package-key.gpg
   ```
9. `yum install kubelet-1.25.3  kubeadm-1.25.3  kubectl-1.25.3 --nogpgcheck -y`
9. 修改 `/etc/containerd/config.toml` 注释`disabled_plugins = [“cri”]`以及添加registry.k8s.io,docker以及自建镜像库
10. install [cri-tools](https://github.com/kubernetes-sigs/cri-tools), 修改`vim /etc/crictl.yaml`, pull pasue3.6 以及tag
11.  kubeadm join .....

## 参考

[通过 kubeadm 安装不同版本的 K8S](https://blog.51cto.com/liqingbiao/5149544)

[kubeadm 极速部署 Kubernetes 1.25 版本集群](https://www.cnblogs.com/gaoyuechen/p/16851566.html)

[使用 kubeadm 搭建 k8s 1.25.3 版本](https://www.cnblogs.com/root0/p/16838234.html)

[K8S 部署 Redis Cluster 集群（三主三从模式） - 部署笔记](https://www.cnblogs.com/kevingrace/p/14412134.html)

[Community Forums,问答论坛](https://discuss.kubernetes.io/)

[How to Tail Kubernetes Logs: Using the Kubectl Command to See Pod, Container, and Deployment Logs](https://sematext.com/blog/tail-kubernetes-logs)

[kubectl 命令参考](https://kubernetes.io/zh-cn/docs/reference/kubectl/)

[kubeadm init phase](https://kubernetes.io/zh-cn/docs/reference/setup-tools/kubeadm/kubeadm-init-phase/)

[Downward API](https://kubernetes.io/zh-cn/docs/concepts/workloads/pods/downward-api/)

[扁平网络 Flannel](https://jimmysong.io/kubernetes-handbook/concepts/flannel.html)

[Multiple databases are not supported on this server; cannot switch to database](https://github.com/StackExchange/StackExchange.Redis/issues/1188)

[net.ipv4.ip_forward=1](https://kubernetes.io/zh-cn/docs/setup/production-environment/container-runtimes/#%E8%BD%AC%E5%8F%91-ipv4-%E5%B9%B6%E8%AE%A9-iptables-%E7%9C%8B%E5%88%B0%E6%A1%A5%E6%8E%A5%E6%B5%81%E9%87%8F)

[调试 DNS 问题](https://kubernetes.io/zh-cn/docs/tasks/administer-cluster/dns-debugging-resolution/)

[解决 Kubernetes 中 Pod 无法正常域名解析问题分析与 IPVS parseIP Error 问题](http://www.mydlq.club/article/78/)

[coredns 官网](https://coredns.io/plugins/forward/)

[Network bridge](https://wiki.archlinux.org/title/Network_bridge)

[Installation Guide](https://kubernetes.github.io/ingress-nginx/deploy/#quick-start)

[防火墙规则](https://blog.csdn.net/yucaifu1989/article/details/104683659)

[ingress-nginx tcp/udp 转发](https://segmentfault.com/a/1190000038582391)，

[真一文搞定 ingress-nginx 的使用](https://cloud.tencent.com/developer/article/1761376)
