---
title: "kubernetes redis"
date: 2022-07-08 09:42:24 +0800
categories: [k8s]
tags: [redis]
---

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: redis
spec:
  containers:
    - name: redis
      image: redis:5.0.4
      command:
        - redis-server
        - "/redis-master/redis.conf"
      env:
        - name: MASTER
          value: "true"
      ports:
        - containerPort: 6379
      resources:
        limits:
          cpu: "0.1"
      volumeMounts:
        - mountPath: /redis-master-data
          name: data
        - mountPath: /redis-master # 将config 对应的文件放入 redis-master文件夹下
          name: config  对应 volumes 中的config
  volumes:
   # 你可以在 Pod 级别设置卷，然后将其挂载到 Pod 内的容器中
    - name: data
      emptyDir: {}
    - name: config
      configMap:
        name: lsd-redis-config
        items:
        - key: redis-config
          path: redis.conf #configmap data.[key] 会生成对应的文件 redis-config

```

```yaml
kind: ConfigMap
apiVersion: v1
metadata:
  name: lsd-redis-config
  namespace: redis
data:
  redis-config: |
    maxmemory 2mb
    maxmemory-policy allkeys-lru
    requirepass xxxx
```


[configmap](https://kubernetes.io/zh-cn/docs/concepts/configuration/configmap/)
