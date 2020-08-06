---
description: >-
  Centos7.x的版本的服务都是以`systemctl start xxxx`来启动的，如何制作自己的开机启动脚本?
  大猪就来带大家如何实现一个自己的开机启动服务。
---

# Centos7 添加开机启动服务

参考 nginx.service

{% code title="/usr/lib/systemd/system/nginx.service" %}
```text
[Unit]
Description=The nginx HTTP and reverse proxy server
After=network.target remote-fs.target nss-lookup.target

[Service]
Type=forking
PIDFile=/run/nginx.pid
# Nginx will fail to start if /run/nginx.pid already exists but has the wrong
# SELinux context. This might happen when running `nginx -t` from the cmdline.
# https://bugzilla.redhat.com/show_bug.cgi?id=1268621
ExecStartPre=/usr/bin/rm -f /run/nginx.pid
ExecStartPre=/usr/sbin/nginx -t
ExecStart=/usr/sbin/nginx
ExecReload=/bin/kill -s HUP $MAINPID
KillSignal=SIGQUIT
TimeoutStopSec=5
KillMode=process
PrivateTmp=true

[Install]
WantedBy=multi-user.target
```
{% endcode %}

说明

```text
[Unit]
Description=描述信息
After=network.target remote-fs.target nss-lookup.target

[Service]
Type=forking#运行方式
PIDFile=PID进程文件
ExecStartPre=开启准备
ExecStart=开启脚本
ExecReload=重启脚本
KillSignal=停止信号量
TimeoutStopSec=停止超时时间
KillMode=杀掉模式
PrivateTmp=独立空间

[Install]
WantedBy=multi-user.target＃脚本启动模式,多用户多网络
```

例子demo

{% code title="/root/user-start.sh" %}
```text
#!/bin/bash
echo "123" > /root/hello.txt
```
{% endcode %}

{% code title="/usr/lib/systemd/system/user-start.service" %}
```text
[Unit]
Description=这是开机启动脚本
After=network.target remote-fs.target nss-lookup.target

[Service]
Type=forking
PIDFile=/run/user-start.pid
ExecStart=/root/user-start.sh
KillSignal=SIGQUIT
TimeoutStopSec=5
KillMode=process
PrivateTmp=true

[Install]
WantedBy=multi-user.target
```
{% endcode %}

设置脚本开始启动

```text
systemctl enable user-start
```

禁止开启启动

```text
systemctl disable user-start
```

常用命令

```text
systemctl start 
systemctl reload
systemctl stop
```



