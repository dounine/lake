---
description: 本文介绍Phabricator使用Docker方式的安装及配置
---

# Phabricator 一键安装及汉化

## 安装使用

下载docker-compose.yml配置文件

```text
$ curl -sSL https://raw.githubusercontent.com/bitnami/bitnami-docker-phabricator/master/docker-compose.yml > docker-compose.yml
```

修改docker-compose.yml

```yaml
version: '2'
services:
  mariadb:
    image: 'bitnami/mariadb:10.3'
    environment:
      - ALLOW_EMPTY_PASSWORD=yes
      - MARIADB_EXTRA_FLAGS=--local-infile=0
    volumes:
      - 'mariadb_data:/bitnami'
  phabricator:
    image: 'bitnami/phabricator:2019'
    ports:
      - '80:80'
      - '443:443'
    volumes:
      - 'phabricator_data:/bitnami'
      - '/root/docker/my_vhost.conf:/opt/bitnami/apache/conf/vhosts/my_vhost.conf'
    environment:
      - PHABRICATOR_PASSWORD=Abc123456
    # 可选配置...
    depends_on:
      - mariadb
volumes:
  mariadb_data:
    driver: local
  phabricator_data:
    driver: local
```

my\_vhost.conf文件

```yaml
<VirtualHost *:80>
  ServerName localhost
  # 可以修改为域名或者IP
  DocumentRoot "/opt/bitnami/phabricator/webroot"
  <Directory "/opt/bitnami/phabricator/webroot">
    Options Indexes FollowSymLinks Includes execCGI
    AllowOverride All
    Require all granted
  </Directory>
   RewriteEngine on
  RewriteRule ^/rsrc/(.*)     -                       [L,QSA]
  RewriteRule ^/favicon.ico   -                       [L,QSA]
  RewriteRule ^(.*)$          /index.php?__path__=$1  [B,L,QSA]
</VirtualHost>
```

在docker-compose.yml文件目录前启动

```yaml
$ docker-compose up -d
```

{% hint style="info" %}
如果启动登录 [http://localhost](http://localhost) 会出现如下类似错误

Site Not Found

This request asked for "/" on host "localhost", but no site is configured which can serve this request.

#### 登录容器并添加如下配置即可解决

$ docker exec -ti docker\_phabricator\_1 bash 

$ /opt/bitnami/phabricator/bin/config set phabricator.base-uri '[http://localhost](http://localhost)'

$ 退出容器然后重启：docker-compose restart
{% endhint %}

可选配置

| 名称 | 描述 | 默认值 |
| :--- | :--- | :--- |
| PHABRICATOR\_HOST | 主机名 | **127.0.0.1** |

## 中文汉化

进入phabricator容器

```bash
$ docker exec -ti docker_phabricator_1 bash
$ cd /opt/bitnami/phabricator/src/extensions
$ curl -O https://raw.githubusercontent.com/arielyang/phabricator_zh_Hans/master/dist/PhabricatorSimplifiedChineseTranslation.php
```

语言页面设置：[http://localhost/settings/user/user/page/language/saved](http://localhost/settings/user/user/page/language/saved)

选择Chinese\(Simplified\)保存即可

## 邮件配置

登录phabricator容器

```bash
$ docker exec -ti docker_phabricator_1 bash
```

配置发送来源

```bash
$ bin/config set metamta.default-address admin@example.com
```

#### 配置smtp

创建mailers.json文件

```bash
cat <<EOF > mailers.json
> [
>   {
>     "key": "stmp-mailer",
>     "type": "smtp",
>     "options": {
>       "host": "smtp.exmail.qq.com",
>       "port": 465,
>       "user": "admin@example.com",
>       "password": "abc123",
>       "protocol": "ssl"
>     }
>   }
> ]
> EOF
```

导入配置

```bash
$ config set cluster.mailers --stdin < mailers.json
```

发送邮件测试

```bash
$ bin/mail send-test --to lake@example.com --subject hello < mailers.json
Reading message body from stdin...
Mail sent! You can view details by running this command:

    phabricator/ $ ./bin/mail show-outbound --id 27
```

#### https设置

登录容器（同上）

设置允许使用https

```bash
$ config set security.require-https true
```

nginx转发配置

```bash
server {
    listen       443 ssl;
    server_name  pha.example.com;
    ssl_certificate /etc/nginx/conf.d/ssl/example.com.pem;
    ssl_certificate_key /etc/nginx/conf.d/ssl/example.com.key;

    location / {
        proxy_pass https://pha.example.com:8002;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
    }
}
server {
    listen 80;
    server_name pha.example.com;
    rewrite ^(.*)$ https://$host$1 permanent;
}
```

