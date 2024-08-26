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
# restart daemonset
kubectl rollout restart  daemonset filebeat -n kube-system
kubectl rollout status ds/filebeat -n kube-system

kubectl create configmap xx_config --from-file=.env
# copy configmap
kubectl get configmap <secret-name> --namespace=<source-namespace> -o yaml \
  | sed 's/namespace: <from-namespace>/namespace: <to-namespace>/' \
  | kubectl create -f -
# Let’s check for the expiration date of a specific certificate
openssl x509 -in /etc/kubernetes/pki/apiserver.crt -noout -text | grep 'Not After'
find /etc/kubernetes/pki/ -type f -name "*.crt" -print | grep -v 'ca.crt$' | xargs -L 1 -t -i bash -c 'openssl x509 -noout -text -in {} | grep "Not After"'
kubeadm certs check-expiration
kubeadm certs renew all
# After renewing the certificates, we need to restart the control plane components, including the API server, controller manager, and scheduler, to use the new certificates.
$ sudo systemctl restart kubelet

$ kubectl -n seatunnel cp ./xxx.jar seatunnel-0:/opt/seatunnel/lib -c seatunnel
# remove all evicted pods
kubectl get pod -n airflow| grep Evicted | awk '{print $1}' | xargs kubectl delete pod -n airflow
```

## yaml block

1. hostAliases
```yaml
  hostAliases:
  - ip: "192.168.1.10"
    hostnames:
    - "foo.local"
    - "bar.local"
```



1. localhost:8080 was refused

   ```bash
   docker ps | grep kube-apiserver

   mkdir -p $HOME/.kube
   sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
   sudo chown $(id -u):$(id -g) $HOME/.kube/config
   ```

   参考[The connection to the server localhost:8080 was refused - did you specify the right host or port?](https://discuss.kubernetes.io/t/the-connection-to-the-server-localhost-8080-was-refused-did-you-specify-the-right-host-or-port/1464)

## kubeadm

### kubeadm init

```bash
# centos
sudo yum install kubelet-1.25.3  kubeadm-1.25.3  kubectl-1.25.3 --nogpgcheck -y

# ubuntu
sudo apt-get update
sudo apt-get install -y  apt-transport-https ca-certificates
curl -fsSL https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-archive-keyring.gpg
curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add
sudo apt install -y kubelet=1.25.3-00  kubeadm=1.25.3-00  kubectl=1.25.3-00
```

为了保证 kubelet 正常工作，你必须禁用交换分区,关闭 selinux 与防火墙,配置 kubernetes /etc/modules-load.d/k8s.conf

```bash
kubeadm init \
--kubernetes-version=v1.25.3 \
--image-repository=registry.aliyuncs.com/google_containers \
--pod-network-cidr=10.244.0.0/16   \
--control-plane-endpoint=kube-apiserver \
--cri-socket unix:///run/containerd/containerd.sock \
--v=5
```

kubeadm init 之前存在给定的证书和私钥对，kubeadm 将不会重写它们

Kubeadm 对集群所有的节点，使用相同的 KubeletConfiguration

执行 init、join 和 upgrade 等子命令会促使 kubeadm 将 KubeletConfiguration 写入到文件 /var/lib/kubelet/config.yaml 中， 继而把它传递给本地节点的 kubelet。

```bash
kubeadm certs check-expiration
```

kubeadm 将 kubelet 配置为自动更新证书

### kubeadm join

```bash
# 生成加入集群命令
kubeadm token create --print-join-command

# 加入集群
kubeadm join kube-apiserver:6443 \
--token i7guj6.at1q41k4bdgi0gbg \
--discovery-token-ca-cert-hash sha256:fc2a4732767762ea293ff2c6da67162bcc27e9850fc4919e769b0215c300d856 \
--cri-socket unix:///run/containerd/containerd.sock \
--v=8

# ubuntu 16.04  kubernetes v1.25.3 需要 install cri-dockerd --cri-socket /run/cri-dockerd.sock


# 将控制平面证书（ca, etcd, api....）上传到集群，并下载解密key
kubeadm init phase upload-certs --upload-certs --v=5

# -certificate-key 下载并解密
kubeadm join kube-apiserver:6443 \
--token 0q5r70.53fggw68nta4697x \
--discovery-token-ca-cert-hash sha256:fc2a4732767762ea293ff2c6da67162bcc27e9850fc4919e769b0215c300d856 \
--certificate-key 27f45c69543808830fb2c59c2dd110a7e010766dd1e5a1ea9a956ef58a194bd0 \
--cri-socket unix:///run/containerd/containerd.sock \
--control-plane \
--v=8

```

###  问题汇总

* pull image

  ```bash
  kubeadm config images list
  kubeadm config images pull --cri-socket=unix:///run/containerd/containerd.sock --image-repository=registry.aliyuncs.com/google_containers --kubernetes-version=v1.25.3
  ```

* cri sandbox image  pause

  pod 所有容器的父容器 kubelet call runsandbox 和 linux namespace  以及 cgroup  申领 network namespace等相关资源配额,pause start 之后将一直阻塞,remove pause first and remove containers in it.

  cri 比如 containerd 以哪个版本的pause作为sandbox可在`/etc/containerd/config.toml`中设置sandbox_image, default `registry.k8s.io/pause`

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

* failed to reserve sandbox name

  apiserver 访问 container runtime interface 失败

  ```bash
  vim /etc/containerd/config.toml

  # disabled_plugins = ["cri"]

  systemctl daemon-reload
  systemctl  restart containerd
  ```

* kubelet invalid slice name

  [kubelet configuration](https://kubernetes.io/docs/reference/command-line-tools-reference/kubelet/)

  > --cgroup-driver string     Default: cgroupfs

  查看 `/var/lib/kubelet/config.yaml`

  > cgroupDriver: systemd

  查看 `/etc/containerd/config.toml` 或 `containerd config default`

  > systemd_cgroup = false

  改为true, restart containerd

  如使用 docker + cri-dockerd 配置cgroupfs，查看`/etc/docker/daemon.json`

  ```json
  {
    "exec-opts": ["native.cgroupdriver=cgroupfs"],
    "insecure-registries" : ["registry.xxxx.com.cn"]
  }
  ```

  restart docker,kubelet

* alter node Role

  `kubectl get no master -o wide --show-labels`

  `kubectl get nodes`

  `kubectl label no ubuntucache-srv node-role.kubernetes.io/control-plane=`

  参考[k8s 中节点 node 的 ROLES 值是 none](https://blog.csdn.net/Lingoesforstudy/article/details/116484624)

* /proc/sys/net/bridge/bridge-nf-call-iptables contents are not set to 1

   执行`echo "1" >/proc/sys/net/bridge/bridge-nf-call-iptables`

   提示  No such file or directory, 没有 bridge folder ，要enable br_netfilter  `modprobe br_netfilter`， 再 `sysctl net.bridge.bridge-nf-call-iptables=1`， 同样 `sysctl net.ipv4.ip_forward=1`

* kubeadm join --control-plane lack of etcd

  [工作流](https://kubernetes.io/zh-cn/docs/reference/setup-tools/kubeadm/kubeadm-join/#join-workflow)

  加入节点有两种方式

  1. 节点发现 （-token + -discovery-token-ca-cert-has + api：port）

    `token` 为共享令牌，`-discovery-token-ca-cert-has` 为验证master ca证书 是否一致（--discovery-token-unsafe-skip-ca-verification 跳过，不建议）， 验证通过后将建立tls引导，提交CSR，签名，控制面返回确定标识，kubeadm 配置kubelet ，通信

    如果是加入控制面， 下载共享证书+ 静态pod清单+ kubeconfig +etcd

  2. 以文件形式提供标准 kubeconfig 文件的一个子集


  分段执行

  `kubeadm join phase kubelet-start --config=config,yaml`

  `kubeadm join phase kubelet-start --token=xx -discovery-token-ca-cert-has=xxxx`

  如果使用 --upload-certs 调用 kubeadm init 命令， 你也可以对控制平面节点调用带 --certificate-key 参数的 join 命令， 将证书复制到该节点。



* failed to publish local member to cluster through raft

  disable firewall

  ```bash
  # ubuntu
  sudo ufw disable

  # centos
  systemctl stop firewalld.service
  ```

* kubeadm reset ,  Failed to unmount mounted directory in /var/lib/kubelet/

  ```bash
  umount <path>
  rm -rf <filePath>
  ```

* cert

  ```bash
  openssl genrsa -out ca.key 2048
  openssl req -x509 -new -nodes -key ca.key -subj "/CN=kube-apiserver" -days 10000 -out ca.crt
  openssl genrsa -out server.key 2048
  ```

## CoreDNS

```bash
kubectl run -it --image=busybox:1.28.3 --rm --restart=Never bash -n kube-system
nslookup  kubernetes.default

:: no server could be reached.

kubectl get ep kube-dns -n kube-system -o json |jq -r ".subsets"
```

coredns 服务/pod 正常，但创建的 pod 无法 nslook 服务，coredns configmap 加 log 也检测不到，反而 master node 上`dig @10.96.0.10 kubernetes.default.svc.cluster.local +noall +answer` 可以找到域名的 ip,无果，删除 dns pod 后自动创建，恢复正常。怀疑与期间修改 flannel subnet 文件以及 cni0 网络相关,尝试 改 ipv4 允许转发,busybox image 应<=1.28.3, 大于此版本可能出现 No Answer 问题

2023-05-23 补充，node2 存在多个 bridge network, ping 192.168.20.xx 时 出现

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

# ip a 或者 ifconfig 或 安装  bridge-utils, 看到bridge network 关联 192.168.16.1
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
sudo tar -zxvf crictl-$VERSION-linux-amd64.tar.gz -C /usr/local/bin
rm -f crictl-$VERSION-linux-amd64.tar.gz
```

```bash
crictl ps
crictl logs <container_id> 2>filename
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

$ crictl config --set runtime-endpoint=unix:///run/containerd/containerd

```


## etcd

由 kubeadm 生成证书，kube-scheduler 调度 kubelet create etcd static pod ,清单见`/etc/kubernetes/manifests/etcd.yaml`

分段执行

`kubeadm init phase etcd local --config=/tmp/${HOST0}/kubeadmcfg.yaml`

config file 参考 [kubeadm Configuration](https://kubernetes.io/docs/reference/config-api/kubeadm-config.v1beta3/#kubeadm-k8s-io-v1beta3-ClusterConfiguration)


### etcdctl连接集群

etcd拓扑结构分stacked(堆叠)和外部

stacked 即 control-plane + etcd，节省资源

donwload [etcdctl](https://github.com/etcd-io/etcd/releases/tag/v3.5.9)

```bash
# 集群成员
/tmp/etcd-download-test/etcdctl \
--endpoints=172.21.14.2xx:2379,kube-apiserverx:2379 \
--cert /etc/kubernetes/pki/etcd/peer.crt \
--key /etc/kubernetes/pki/etcd/peer.key \
--cacert /etc/kubernetes/pki/etcd/ca.crt \
member list

# 集群健康
/tmp/etcd-download-test/etcdctl \
--endpoints=172.21.14.2xx:2379,kube-apiserverx:2379 \
--cert /etc/kubernetes/pki/etcd/peer.crt \
--key /etc/kubernetes/pki/etcd/peer.key \
--cacert /etc/kubernetes/pki/etcd/ca.crt \
endpoint health

# 移除成员
/tmp/etcd-download-test/etcdctl \
--endpoints=172.21.14.2xx:2379,kube-apiserverx:2379 \
--cert /etc/kubernetes/pki/etcd/peer.crt \
--key /etc/kubernetes/pki/etcd/peer.key \
--cacert /etc/kubernetes/pki/etcd/ca.crt \
member remove ${MEMBER_ID}

# 添加成员
/tmp/etcd-download-test/etcdctl \
--endpoints=172.21.14.2xx:2379,kube-apiserverx:2379 \
--cert /etc/kubernetes/pki/etcd/peer.crt \
--key /etc/kubernetes/pki/etcd/peer.key \
--cacert /etc/kubernetes/pki/etcd/ca.crt \
member add ${hostname} --peer-urls=https://${ip}:2380
```

join master 过程中出现 etcd 出现 no leader 问题 和 tls connecion refused ,与证书有关，不要轻易改动 /etc/kubernetes/pki 目录，改的话要先备份。

假设集群不健康，无法通过常规方式restart，可尝试命令行的方式单独run etcd as a new cluster,风险大

```bash
# Force to create a new one-member cluster.
--force-new-cluster 'true'
```

利用备份文件和 etcd.yaml 在 kubernetes 外重新搭建 etcd 实例,不验证 auth，apiserver 的 yaml 文件中 etc listen url 也相应改为 http://127.0.0.1:2379

```bash
/tmp/etcd-download-test/etcd  \
--listen-client-urls=http://127.0.0.1:2379   \
--advertise-client-urls=http://127.0.0.1:2379  \
--client-cert-auth=false \
--peer-client-cert-auth=false  \
--name=master \
--initial-cluster=master=http://127.0.0.1:2380 \
--initial-advertise-peer-urls=http://127.0.0.1:2380  \
--data-dir=/var/lib/etcd_bak \
--initial-cluster-state=new \
--force-new-cluster='true'
```
### 参考

[手动建 etcd 集群](https://kubernetes.io/zh-cn/docs/setup/production-environment/tools/kubeadm/setup-ha-etcd-with-kubeadm/)

[Configuration options](https://etcd.io/docs/v3.5/op-guide/configuration/)

[How to Add and Remove Members](https://etcd.io/docs/v3.5/tutorials/how-to-deal-with-membership/)

[如何安装 etcd 和 etcdctl](https://github.com/etcd-io/etcd/releases/)


##  runc

```bash
wget https://github.com/opencontainers/runc/releases/download/v1.1.4/runc.amd64
sudo install -m 755 runc.amd64 /usr/local/sbin/runc
```


## cni

容器运行时必须配置为加载所需的 CNI 插件，从而实现 Kubernetes 网络模型

docker default cni-plugin is bridge

查看 `/etc/cni/net.d/`

###  cni-plugin

```bash
sudo mkdir -p /opt/cni/bin/
sudo wget https://github.com/containernetworking/plugins/releases/download/v1.1.1/cni-plugins-linux-amd64-v1.1.1.tgz
sudo tar Cxzvf /opt/cni/bin cni-plugins-linux-amd64-v1.1.1.tgz
```

* flannel plugin

  install

  ```bash
  curl -o kube-flannel.yml https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml
  kubectl apply -f kube-flannel.yml
  docker pull quay.io/coreos/flannel:v0.14.0
  kubectl apply -f kube-flannel.yml
  ```
  k8s create daemonset in node,相当于install flannel-cni-plugin , 查看`/run/flannel/subnet.env`和`/etc/cni/net.d`


  > Flannel.1: overlay网络的设备，用来进行 vxlan 报文的处理（封包和解包）。不同node之间的pod数据流量都从overlay设备以隧道的形式发送到对端。

  > Cni0 :网桥设备，每创建一个pod都会创建一对 veth pair。其中一端是pod中的eth0，另一端是Cni0网桥中的端口（网卡）。Pod从网卡eth0发出的流量都会发送到Cni0网桥设备的端口（网卡）上。

  > pod发数据，发到cni0, 再到flannel，隧道形式发送对端flannel，flannel 解释数据 ，发cni0, 再到pod, `ip route`查看

  参考

  [扁平网络 Flannel](https://jimmysong.io/kubernetes-handbook/concepts/flannel.html)

  [kubernetes之flannel 网络分析](https://zhuanlan.zhihu.com/p/340747753)


### 问题汇总

* open /run/flannel/subnet.env no such file or directory

  新建 /run/flannel/subnet.env 文件

  ```json
  FLANNEL_NETWORK=10.244.0.0/16
  FLANNEL_SUBNET=10.244.0.1/24
  FLANNEL_MTU=1450
  FLANNEL_IPMASQ=true
  ```

  kube-flannel pod 挂载了`/run/flannel` ，不同的 node，FLANNEL_SUBNET 会有所不同`10.244.1.1/24,10.244.2.1/24`

* k8s 安装 flannel 报错“node "master" pod cidr not assigned”

   Kubeadm Init 的时候，没有增加 --pod-network-cidr 10.244.0.0/16

   kubectl patch node gz-vmtest -p '{"spec":{"podCIDR":"10.244.5.0/16"}}'

* apply pod 时出现 `failed to set bridge addr: "cni0" already has an IP address different from 10.244.1.1/24`

   del cni0 flannel1.1 ,stop & remove kube-flannel pod, restart

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

## CRI 容器运行时

kubelet 通过 cri-socket 调用 (早前dockershim 1.24后弃用，没有实现CRI标准)cri-dockerd/containerd  create sandbox, run containers in it.

如果单独install containerd.io , 需install runc（cgroup 相关）以及 cni(k8s 有default )
### containerd

`containerd plugin ls` 列出 containerd 插件


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
version = 1

[plugins]
  [plugins.cri]
    [plugins.cri.registry]
        [plugins.cri.registry.mirrors]
          [plugins.cri.registry.mirrors."registry.k8s.io"]
            endpoint = ["https://registry.cn-hangzhou.aliyuncs.com/google_containers"]
          [plugins.cri.registry.mirrors."docker.io"]
            endpoint = ["https://registry.cn-hangzhou.aliyuncs.com"]

```

```yaml
version = 2

[plugins."io.containerd.grpc.v1.cri"]
  sandbox_image = "http://registry.aliyuncs.com/google_containers/pause:3.8"
[plugins."io.containerd.grpc.v1.cri".registry]
  [plugins."io.containerd.grpc.v1.cri".registry.mirrors]
    [plugins."io.containerd.grpc.v1.cri".registry.mirrors."docker.io"]
      endpoint = ["https://registry.docker-cn.com"]
    [plugins."io.containerd.grpc.v1.cri".registry.mirrors."registry.k8s.io"]
      endpoint = ["http://registry.aliyuncs.com/google_containers"]
    [plugins."io.containerd.grpc.v1.cri".registry.mirrors."registry.lesaunda.io"]
      endpoint = ["http://192.168.1.193:5000"]
```


containerd v1.2 前 使用 [配置语法](https://github.com/containerd/cri/blob/release/1.2/docs/registry.md)

1.3之后使用 [配置语法](https://github.com/containerd/containerd/blob/main/docs/cri/registry.md)




如阿里云timeout,参考[这里](https://www.cnblogs.com/boonya/p/15954368.html),改 http://hub-mirror.c.163.com


`systemctl daemon-reload`
`systemctl enable --now containerd` 守护进程
`systemctl restart containerd`

### docker engine

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

Kubernetes control plane is running at https://kube-apiserver:6443
CoreDNS is running at https://kube-apiserver:6443/api/v1/namespaces/kube-system/services/kube-dns:dns/proxy

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
## 污点 node

查看节点污点

`kubectl get nodes -o=custom-columns=NodeName:.metadata.name,TaintKey:.spec.taints[*].key,TaintValue:.spec.taints[*].value,TaintEffect:.spec.taints[*].effect`

NodeName            TaintKey                                TaintValue   TaintEffect
gz-k8ssrv-master1   <none>                                  <none>       <none>
lsd-server-01       <none>                                  <none>       <none>
node1               node-role.kubernetes.io/control-plane   <none>       NoSchedule
node2               node-role.kubernetes.io/control-plane   <none>       NoSchedule
ubuntucache-srv     node-role.kubernetes.io/control-plane   <none>       NoSchedule

`kubernetes 提示 1 node(s) had taints that the pod didn't tolerate` 表示 节点污点，pod不能容忍，所以不部署在此节点

> NoSchedule,The Kubernetes scheduler will only allow scheduling pods that have tolerations for the tainted nodes.
> PreferNoSchedule,The Kubernetes scheduler will try to avoid scheduling pods that don’t have tolerations for the tainted nodes.
> NoExecute,Kubernetes will evict the running pods from the nodes if the pods don’t have tolerations for the tainted nodes.


```bash
kubectl taint nodes node1 key1=value1:NoSchedule
# remove
kubectl taint nodes node1 key1=value1:NoSchedule-
```

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx
  labels:
    env: test
spec:
  containers:
  - name: nginx
    image: nginx
    imagePullPolicy: IfNotPresent
  tolerations:
  - key: "example-key"
    operator: "Exists"
    effect: "NoSchedule"
```

pod 将在 存在污点 key=example-key 而且 effect = NoSchedule 的节点上部署

operator 的默认值是 Equal

`kubectl taint nodes --all node-role.kubernetes.io/master-`

`kubectl label nodes ubuntucache-srv node.kubernetes.io/exclude-from-external-load-balancers=`

`kubectl taint node ubuntucache-srv node-role.kubernetes.io/control-plane=:NoSchedule`

非体面关闭时 ，statefulSet 无法创建同名pod，要手动添加污点，分离卷操作

`kubectl taint node ubuntucache-srv node.kubernetes.io/out-of-service=:NoSchedule`

在添加 node.kubernetes.io/out-of-service 污点之前， 应该验证节点已经处于关闭或断电状态（而不是在重新启动中）。

将 Pod 移动到新节点后，用户需要手动移除停止服务的污点， 并且用户要检查关闭节点是否已恢复，因为该用户是最初添加污点的用户。


### 参考

[kubernetes 提示 1 node(s) had taints that the pod didn't tolerate](https://www.cnblogs.com/ip99/p/14231203.html)

[污点](https://kubernetes.io/zh-cn/docs/concepts/scheduling-eviction/taint-and-toleration/)

[Kubernetes Taints & Tolerations](https://www.densify.com/kubernetes-autoscaling/kubernetes-taints/)


## nodeSelector

node 打label

```bash
kubectl label node1 dedicated=test
# remove lable
kubectl label node1 dedicated-
```

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx
  labels:
    env: test
spec:
  containers:
  - name: nginx
    image: registry.lesaunda.com.cn/busybox
    command:
      - sleep
      - "3600"
    imagePullPolicy: IfNotPresent
  nodeSelector:
    dedicated: test
```


## Tools

```bash
yum -y install net-tools
```

##  PVC

nfs 卷能将 NFS (网络文件系统) 挂载到你的 Pod 中。 不像 emptyDir 那样会在删除 Pod 的同时也会被删除，nfs 卷的内容在删除 Pod 时会被保存，卷只是被卸载。

若 pvc status = Terminating

打补丁

`kubectl -n redis-cluster patch pvc data-redis-cluster-0  -p '{"metadata":{"finalizers":null}}'`


## ingrees-nginx

### 部署

`kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.5.1/deploy/static/provider/cloud/deploy.yaml`

利用pod sepc 反亲和性 确保每个node只有一个ingress-nginx

引用  [Question: nginx - multiple replicas or daemonSet?](https://github.com/kubernetes/ingress-nginx/issues/875)

> Using a deployment with an anti-affinity rule to avoid multiple replicas in the same node is, in most of the cases, more than enough.

以及参考 [将 Pod 指派给节点](https://kubernetes.io/zh-cn/docs/concepts/scheduling-eviction/assign-pod-node/#inter-pod-affinity-and-anti-affinity)

```yaml
affinity:
  podAntiAffinity:
    preferredDuringSchedulingIgnoredDuringExecution:
    - weight: 100
      podAffinityTerm:
        labelSelector:
          matchExpressions:
          - key: app.kubernetes.io/name
            operator: In
            values:
            - ingress-nginx
        topologyKey: "kubernetes.io/hostname"
```

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

所有pods down, 要手动delete rdb file, aof file, nodes.conf ，redis-cli 执行 flushdb, cluster reset,比较复杂，采用下面方式重建

如果所有 pod 下线，需要重新建群

```sh
$ kubectl delete -f redis-cluster.yml # 删除资源
$ for i in {0..5}; do kubectl delete pvc/data-redis-cluster-$1 -n redis-cluster; done; # 删除pvc
$ for i in {0..5}; do rm -rf /root/nfs_root/redis-cluster-data-redis-cluster-$i; done; # 删除nfs
$ kubectl apply -f redis-cluster.yml # 部署资源
$ kubectl get pods -l app=redis-cluster -n redis-cluster -o jsonpath='{range.items[*]}{.status.podIP}:6379 ' # 获取ip
$ kubectl -n redis-cluster exec -it redis-cluster-0 /bin/sh # 进入pod
$ redis-cli -a password --cluster create 10.244.2.89:6379 10.244.2.90:6379 10.244.0.20:6379 10.244.2.91:6379 10.244.0.21:6379 10.244.2.92:6379 --cluster-replicas 1 # build cluster
```

或者(验证过，无法build cluster,要 delete rdb file, aof file, nodes.conf ，redis-cli 执行 flushdb, cluster reset)

1. `kubectl delete -f redis-cluster.yml` 删除资源
2. `rm -f /ifs/kubernetes/redis-cluster-data-redis-cluster-0*/nodes.conf*` 逐一删除 nodes.conf
3. `kubectl apply -f redis-cluster.yml` 重新建立集群

for i in {0..5}; do rm -rf /root/nfs_root/redis-cluster-data-redis-cluster-$i; done;

for i in {0..5}; do kubectl delete pvc/data-redis-cluster-$1 -n redis-cluster; done;

### redisinsight

参考[Install RedisInsight on Kubernetes](https://docs.redis.com/latest/ri/installing/install-k8s/)

[LoadBalancer](https://kubernetes.io/zh-cn/docs/concepts/services-networking/service/#loadbalancer)

> 要实现 type: LoadBalancer 的服务，Kubernetes 通常首先进行与请求 type: NodePort 服务等效的更改。 cloud-controller-manager 组件然后配置外部负载均衡器以将流量转发到已分配的节点端口。

访问 http://172.21.14.208:30738/


##  mysql cluster

[Install using Manifest Files](https://dev.mysql.com/doc/mysql-operator/en/mysql-operator-installation-kubectl.html)


> start failed in pod mysqlcluster-2_mysql(7059d32c-bfd3-4d75-9155-5cb1c1dcc2bc): ErrImagePull: rpc error: code = Unknown desc = failed to pull and unpack image "container-registry.oracle.com/mysql/community-operator:8.0.33-2.0.10": failed to read expected number of bytes: unexpected EOF

```bash
ctr -n k8s.io i pull container-registry.oracle.com/mysql/community-operator:8.0.33-2.0.10
# 部分layer 下载非常慢，在其他node push image，再pull,同样慢，layer 指定了url，去掉 -n k8s.io ，下载再加上即可

# 删除 会stuck ，+ force
kubectl -n mysql delete pod mysqlcluster-2 --force

systemctl restart kubelet
```


###  修改时区

[MySQL Set UTC time as default timestamp](https://dba.stackexchange.com/questions/20217/mysql-set-utc-time-as-default-timestamp)

```sql
SELECT @@global.time_zone, @@session.time_zone;
SELECT CURRENT_TIMESTAMP();
SET @@session.time_zone='+08:00';
```
集群-永久性生效

`kubectl  -n mysql edit  cm mysqlcluster-initconf`

```yaml
mycnf: |
  [mysqld]
  max_connections=162
  default_time_zone='+08:00'
```

`kubectl -n mysql scale statefulSet mysqlcluster --replicas=0`

`kubectl -n mysql scale statefulSet mysqlcluster --replicas=3`


## 修改 Control-Plane-Endpoint

api service 做 HA 高可用

```bash
kubeadm init \
--kubernetes-version=v1.25.3 \
--image-repository=registry.aliyuncs.com/google_containers \
--pod-network-cidr=10.244.0.0/16  \
--control-plane-endpoint=<LB address or commonname> \
--cri-socket=unix:///run/containerd/containerd.sock \
--v=9
```

如果想从non-HA Convert to HA ,非常复杂，可以参考 [Converting Kubernetes to an HA Control Plane](https://blog.scottlowe.org/2019/08/12/converting-kubernetes-to-ha-control-plane/)

保留etcd ，reinit 的方法不可行，将丢失init的node，考虑将ip指向vm，再vm上做LB


##  Install as node

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
   ```bash
   # ubuntu
   sudo sed -i.bak '/ swap / s/^\(.*\)$/#\1/g' /etc/fstab
   sudo systemctl mask swap.target
   ```
5. swapoff -a #关闭内存交换,同时修改 /etc/fstab 注释 swap 行
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
9.  `yum install kubelet-1.25.3  kubeadm-1.25.3  kubectl-1.25.3 --nogpgcheck -y`
10. 修改 `/etc/containerd/config.toml` 注释`disabled_plugins = [“cri”]`以及添加registry.k8s.io,docker以及自建镜像库
11. install [cri-tools](https://github.com/kubernetes-sigs/cri-tools), 修改`vim /etc/crictl.yaml`, pull pasue3.6 以及tag
12.  kubeadm join .....


##  container-runtime
> Flag --container-runtime has been deprecated, will be removed in 1.27 as the only valid value is 'remote'
`vim  /var/lib/kubelet/kubeadm-flags.env`  移除 `--container-runtime`参数

##  Helm

Helm 是 Kubernetes 的包管理器

install

```sh
$ curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3
$ chmod 700 get_helm.sh
$ ./get_helm.sh
```

[安装Helm](https://helm.sh/zh/docs/intro/install/)

```sh
helm add repo [name] [url]

helm search repo [name]
```

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

