---
title: "Ubuntu Commands"
date:  2021-12-07 10:53:16 +0800
categories: [系统]
tags: [Ubuntu]
---

## 1. daemon
As explained above, a daemon is a non-interactive program. It runs all the time, and it’s not connected to the terminal. Even when you close the terminal, the operating system will not stop the daemon as it will run in the background.

On the other hand, a process will stop when the terminal closes because it is an executing program instance.

[What is a Daemon?](https://www.liquidweb.com/kb/what-is-a-daemon/)


[apt update 和 apt upgrade 有什么区别](https://embeddedinventor.com/apt-update-apt-upgrade-command-explained-for-beginners/)

[How To Fix “E: Could not get lock /var/lib/dpkg/lock” Error On Ubuntu](https://ostechnix.com/how-to-fix-e-could-not-get-lock-var-lib-dpkg-lock-error-on-ubuntu/)



[合盖不休眠](https://blog.csdn.net/xiaoxiao133/article/details/82847936)
[不休眠]https://askubuntu.com/questions/47311/how-do-i-disable-my-system-from-going-to-sleep
[APT vs APT-GET: What's the Difference?](https://phoenixnap.com/kb/apt-vs-apt-get)
[Getting started with Docker: Running an Ubuntu Image](https://dev.to/netk/getting-started-with-docker-running-an-ubuntu-image-4lk9#:~:text=To%20exit%20the%20container%20simply%20type%20exit%20from,Docker%20console%20when%20you%20created%20the%20Ubuntu%20container.)
[docker run image ](https://dev.to/netk/getting-started-with-docker-running-an-ubuntu-image-4lk9)
[Copy file](https://www.cyberciti.biz/faq/ubuntu-copy-file-command/)
[简单性能监控](https://www.howtoforge.com/tutorial/ubuntu-performance-monitoring/#:~:text=How%20to%20monitor%20your%20system%20performance%20on%20%28Ubuntu%29,type%20%E2%80%9Ctop%E2%80%9D%20and%20hit%20enter.%203%20Lm-sensors.%20)

[ls -ltr](https://askubuntu.com/questions/640746/difference-between-ls-l-ls-ltr-and-ll)

[学习常用的命令](https://vitux.com/40-most-used-ubuntu-commands/)
[ The Directory Tree](https://help.ubuntu.com/lts/installation-guide/armhf/apcs02.html)

## 2. 命令
### 2.1. netstatus

`netstat -tlnp`

`apt-get install net-Tools`

端口 53 systemd-resolved为域名系统 (DNS)（包括DNSSEC和DNS over TLS）、多播 DNS (mDNS)和链路本地多播名称解析 (LLMNR)提供解析器服务。

端口 631 打印服务

[link](https://www.howtogeek.com/513003/how-to-use-netstat-on-linux/)

### 2.2. 其他

- sudo  = superuser do
- su = switch user
- sudo adduser user_name
- groups
- id user_name
- sudo usermod -aG sudo user_name

### 2.3. 修改文件权限

查看文件权限 ls -l

修改

chmod ugo/a+rw filename

u = owner 拥有者
g = group 所属组
o = other 其他
a = all

或者chown

chown username filename

- npoc/lscpu 核数
- netstat 防火墙
- top/htop 监控
- rm/mv/cp sourcefile targetfile 删除 移动 复制
- cat filename | grep "searchstring" | wc -l (count line)

###  2.4. dig

dig是Domain Information Groper的首字母缩写词

## 3. 参考

[API查询](https://wiki.archlinux.org/title/Man_page)

[How to Use the dig Command on Linux](https://www.howtogeek.com/663056/how-to-use-the-dig-command-on-linux/)



