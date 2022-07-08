---
 title: "kubernetes rabbitmq"
 date:  2022-07-08 11:44:57 +0800
 categories: [k8s]
 tags: [rabbit]
---


## 部署

`kubectl apply -f https://github.com/rabbitmq/cluster-operator/releases/latest/download/cluster-operator.yml`

`kubectl apply -f https://raw.githubusercontent.com/rabbitmq/cluster-operator/main/docs/examples/hello-world/rabbitmq.yaml`


## 用户密码

查看default account 和 password

`username="$(kubectl get -n rabbitmq-system secret lsd-rabbitmq-default-user -o jsonpath='{.data.username}' | base64 --decode)"`

`password="$(kubectl get -n rabbitmq-system secret lsd-rabbitmq-default-user -o jsonpath='{.data.password}' | base64 --decode)"`

添加用户

`kubectl exec hello-world-server-0 -c rabbitmq -- rabbitmqctl authenticate_user default_user_1-S8lZU9WCaTIxphE3k guest`


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

## 参考

[RabbitMQ Cluster Kubernetes Operator](https://github.com/rabbitmq/cluster-operator)

[端口转发](https://minikube.sigs.k8s.io/docs/tutorials/nginx_tcp_udp_ingress/)

[RabbitMQ Cluster Operator for Kubernetes](https://www.rabbitmq.com/kubernetes/operator/operator-overview.html)

