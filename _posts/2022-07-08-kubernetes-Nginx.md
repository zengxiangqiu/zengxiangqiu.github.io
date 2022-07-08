---
title: "kubernetes Nginx"
date:  2022-07-08 12:02:20 +0800
categories: [k8s]
tags: [nginx]
---


查看 pod ingress-nginx-controller  nginx.conf

`$(kubectl get pod -o go-template --template '{{range .items}}{{.metadata.name}}{{"\n"}}{{end}}' -n ingress-nginx -l app=app.kubernetes.io/component: controller) -n ingress-nginx cat /etc/nginx/nginx.conf`

```conf
stream {
  upstream rabbitmq {
    server 192.168.xx.x:5672;
  }

  server {
    listen 5672;
    proxy_pass rabbitmq;
    proxy_connect_timeout 1s;
    proxy_timeout 3s;
    proxy_set_header   Host   $host;
    proxy_set_header   X-Real-IP  $remote_addr;
    proxy_set_header   X-Forwarded-For $proxy_add_x_forwarded_for;
  }
}

server {
        listen 80;
        server_name www.lsd.com;
        location / {
          proxy_pass http://192.168.xx.x/lsd/;
        }
}
server {
        listen 15672;
        server_name lsd-server-01;
        location / {
          proxy_pass http://192.168.xx.x:15672;
        }
}

```

rabbitmq management 15672  nginx 代理

```conf
upstream rabbitbackend {
    server 127.0.0.1:15672;
}


server {
    listen  80;
    server_name rabbit.erps.sunnyb2b.com;
    fastcgi_connect_timeout 600;
    fastcgi_send_timeout 600;
    fastcgi_read_timeout 600;

    location / {
        port_in_redirect on;
        proxy_redirect off;
        proxy_pass http://rabbitbackend;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header User-Agent $http_user_agent;
        proxy_set_header Host $http_host;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }

    location ~* /rabbitmq/api/ {
        rewrite ^ $request_uri;
        rewrite ^/rabbitmq/api/(.*) /api/$1 break;
        return 400;
        proxy_pass http://rabbitbackend$uri;
        proxy_buffering                    off;
        proxy_set_header Host              $http_host;
        proxy_set_header X-Real-IP         $remote_addr;
        proxy_set_header X-Forwarded-For   $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }

}
```


## Command

`sudo vim /etc/nginx/nignx.conf`

`sudo nginx -s reload`

`cat /var/log/nginx/access.log`

`cat /var/log/nginx/error.log`

