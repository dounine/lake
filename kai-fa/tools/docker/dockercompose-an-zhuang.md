---
description: 介绍使用命令的方式一键安装 docker-compose
---

# docker-compose 安装

下载docker-compose可执行文件

```text
$ sudo curl -L "https://github.com/docker/compose/releases/download/1.24.1/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
```

添加可执行权限

```text
$ sudo chmod +x /usr/local/bin/docker-compose
```

检查是否成功安装

```text
docker-compose --version
```

