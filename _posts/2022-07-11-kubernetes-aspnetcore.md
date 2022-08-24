---
title: "kubernetes aspnetcore"
date:  2022-07-11 14:55:22 +0800
categories: [DevOps]
tags: [k8s]
---


`kubectl create configmap lsd-inventory-appsettings --from-file=appsettings.json=./appsettings.json -n lsd`


[.NET Core 使用 K8S ConfigMap的正确姿势](https://www.cnblogs.com/sheng-jie/p/Using-ConfigMap-with-NETCore.html)



## TroubleShot

serilog 记录的时间TimeStamp是UTC，可以再docker build image 时指定 timezone

```dockerfile
ENV TZ=Asia/Shanghai
ENV DEBIAN_FRONTEND=noninteractive
```


restsharp get 没问题，put 出现 基础连接已关闭

[win7 enable TLS 1.2 ](https://support.microsoft.com/en-us/topic/update-to-enable-tls-1-1-and-tls-1-2-as-default-secure-protocols-in-winhttp-in-windows-c4bd73d2-31d7-761e-0178-11268bb10392)

怀疑是win7 tls 1.2 没有开启或者没有打补丁


[How to Handle Timezones in Docker Containers](https://www.howtogeek.com/devops/how-to-handle-timezones-in-docker-containers/)
