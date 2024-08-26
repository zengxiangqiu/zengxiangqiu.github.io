---
title: "kubernetes volume"
date:  2023-05-17 10:28:14 +0800
categories: [kubernetes]
tags: [volume]
---
## emptyDir

[emptyDir](https://kubernetes.io/zh-cn/docs/concepts/storage/volumes/#emptydir)

用于节点临时存储，pod消失，volume内容消失

## pv持久卷


分静态制备和动态制备

###  静态制备

pv 创建，status available pending, 等待绑定，pvc创建绑定，已申领未绑定， pod 创建，persistentVolumeClaim 中 绑定对应的pvc进行访问

创建-申领-绑定

###  动态制备

创建StorageClass 存储类, 创建serviceaccount, rbac , role, rolebinding等，部署创建provisioner pod（serviceaccount, storageclass）, 通过storageclass 创建pvc, 再部署其他需要动态制备持久卷的容器

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: nfs-elastic
spec:
  accessModes:
    - ReadWriteMany
  storageClassName: "nfs-gzk8s-storage" # 动态制备，对应storagclassname ,静态对应控制符串
  resources:
    requests:
      storage: 500Gi
  # volumeName: foo-pv  # 对应之前静态制备的pv卷name，动态制时需注释掉！
```


other deployment

```yaml
volumes:
  - name: storage
      persistentVolumeClaim:
        claimName: nfs-elastic
```

##  访问模式

ReadWriteOnce 卷可以被一个节点以读写方式挂载。 ReadWriteOnce 访问模式也允许运行在同一节点上的多个 Pod 访问卷。

ReadWriteMany 卷可以被多个节点以读写方式挂载

## 容量

单位是 Ki (kibi)、Mi (mebi)、Gi (gibi)、 Ti (tebi)、 Pi (pebi)、 Ei (exbi)

## 回收策略

- Retain -- 手动回收
- Recycle -- 基本擦除 (rm -rf /thevolume/*)
- Delete -- 诸如 AWS EBS、GCE PD、Azure Disk 或 OpenStack Cinder 卷这类关联存储资产也被删除

user 删除 PVC 不会立刻移除，会等待关联的pod退出， 同理，删除PV要求没有bound的PVC，顺序上是先terminal pod，delete pvc ,自动 delete pv

## nfs install

1. server

```sh

yum -y install rpcbind nfs-utils

systemctl enable --now nfs-server.service

systemctl status rpcbind

# 服务未开启，则开启

# 创建path
mkdir -m 0755  -p /data/nfs

# 默认只有50G 挂载在root下
df -hT /data/nfs
# 修改为home下，容量大
mount -t auto /dev/mapper/centos-home /data/nfs

# 设置权限
echo "/data/nfs    *(rw,sync,no_root_squash)" >> /etc/exports

# 生效
exports -rv

# 必要时关闭防火墙
systemctl  stop  firewalld.service

# 查看nfs 公布的path

showmount -e
```

2. client

[在客户端测试nfs](https://www.kuboard.cn/learning/k8s-intermediate/persistent/nfs.html#%E5%9C%A8%E5%AE%A2%E6%88%B7%E7%AB%AF%E6%B5%8B%E8%AF%95nfs) 用于测试server，非必需

storageclass 会指定provisioner(制备器)，比如nfs需要外部制备器（docker.io/dyrnq/nfs-subdir-external-provisioner:v4.0.2）


## 参考

[Configure NFS as Kubernetes Persistent Volume Storage](https://computingforgeeks.com/configure-nfs-as-kubernetes-persistent-volume-storage/)

[Kubernetes PVC Guide: Basics, Tutorials and Troubleshooting Tips](https://komodor.com/learn/kubernetes-pvc-guide-basic-tutorial-and-troubleshooting-tips/)

[kubernetes examples](https://github.com/kubernetes/examples/blob/master/staging/volumes/nfs/nfs-busybox-deployment.yaml)

[linux 分区](https://phoenixnap.com/kb/linux-create-partition)


[pvc capacity just a label](https://github.com/kubernetes/kubernetes/issues/48701)
