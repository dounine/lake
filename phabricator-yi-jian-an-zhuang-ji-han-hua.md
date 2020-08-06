---
description: 本文介绍Phabricator使用Docker方式的安装及配置wget
---

# Phabricator 一键安装及汉化

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

### 中文汉化

进入phabricator容器

```bash
docker exec -ti docker_phabricator_1 bash
cd /opt/bitnami/phabricator/src/extensions
curl -O https://raw.githubusercontent.com/arielyang/phabricator_zh_Hans/master/dist/PhabricatorSimplifiedChineseTranslation.php
```

语言页面设置：[http://localhost/settings/user/user/page/language/saved](http://localhost/settings/user/user/page/language/saved)



