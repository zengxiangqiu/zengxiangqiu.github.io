---
title: "AspnetCore Docker部署https"
date:  2022-03-07 15:05:33 +0800
categories: [AspnetCore]
tags: [部署]]
---

## 1. 证书

在本机Ubuntu/Windows利用OpenSSL生成自签名的CA root证书(Ubuntu认crt格式)，
利用该证书签发IdentityServer和api项目证书（apsnetcore认pfx格式）
生成证书及转换见[OpenSSL教程](/HTTPS)
分别将ca证书和pfx挂载到对应的docker容器中，并让容器信任该ca证书

如果出现partialchain问题，可以进入容器`apt-get update`安装curl`apt-get install curl`执行检查`curl -v IP：prot`

## 2. 网络

docker-compose 文件中可以设置networks,实现网络隔离

同一网络下如何访问：

`https://container—name:port`

在IdentityServer中修改IssuerUri，api中修改Authority

docker命令为容器添加网络 `docker networks connect network_name container_name`

## 3. MySql

抛出异常 `mbind: Operation not permitted`

设置docker环境参数cap_add = SYS_NICE

另外要设置密码 MYSQL_ROOT_PASSWORD=1234

如何设置远程访问，进入容器 `docker exec -it mysql_databse /bin/bash`，进入mysql`mysql`,切换数据库`use mysql`，修改root的host`update user set host ='%' where user='root';`执行`flush privileged`

远程访问MySQL容器命令： `mysql -h IP -u root -p `

## 4. 自签名证书

### 4.1. Ubuntu

```
$ openssl req -x509 \
    -newkey rsa \
    -outform PEM -out tls-rootca.pem \
    -keyform PEM -keyout tls-rootca.key.pem \
    -days 35600 \
    -nodes \
    -subj "/C=cn/O=mycomp/OU=mygroup/CN=rootca"

# 查看证书
$ openssl x509 -text -noout -in tls-rootca.pem

$ openssl req -newkey rsa:2048 \
  -outform PEM -out tls-intermca.csr \
  -keyform PEM -keyout tls-intermca.key.pem \
  -nodes \
  -extensions v3_ca \
  -config /etc/pki/tls/openssl.cnf \
  -subj "/C=cn/O=mycomp/OU=mygroup/CN=intermca"

$ openssl x509 \
  -req -days 365 \
  -in tls-intermca.csr \
  -out tls-intermca.pem \
  -CA tls-rootca.pem \
  -CAkey tls-rootca.key.pem \
  -CAcreateserial \
  -extensions v3_ca \
  -extfile /etc/pki/tls/openssl.cnf
```

extfile = /usr/lib/ssl/openssl.cnf

具体可参考 [使用openssl创建自签名的证书链](https://www.jianshu.com/p/16fd25999c32)

### 4.2. windows

PowerShell脚本

具体可参考考 [在本地启用 HTTPS 在 Docker 上运行 IdentityServer4 时保护 API](https://mjarosie.github.io/dev/2020/09/24/running-identityserver4-on-docker-with-https.html)

