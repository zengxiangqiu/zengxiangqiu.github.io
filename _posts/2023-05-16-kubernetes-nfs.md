---
title: "kubernetes nfs"
date:  2023-05-16 14:29:02 +0800
categories: [k8s]
tags: [k8s,nfs]
---

在所有节点安装nfs client

yum install -y nfs-utils


```sh
nfsstat -s -c
```


[Using nfsstat and nfsiostat to troubleshoot NFS performance issues on Linux](https://www.redhat.com/sysadmin/using-nfsstat-nfsiostat)
