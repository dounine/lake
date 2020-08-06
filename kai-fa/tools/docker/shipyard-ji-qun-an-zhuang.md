---
description: 如何使用`Shipyard`安装一个`docker`的集群，以下为此文安装过程，使用centos7作为测试系统。
---

# Shipyard集群安装

docker 安装可参考官方安装说明 [docker install](https://docs.docker.com/)

安装`Datastore`帐号密码管理容器

```text
docker run \
    -ti \
    -d \
    --restart=always \
    --name shipyard-rethinkdb \
    rethinkdb
```

安装集群发现`Discovery`服务

```text
docker run \
    -ti \
    -d \
    -p 4001:4001 \
    -p 7001:7001 \
    --restart=always \
    --name shipyard-discovery \
    microbox/etcd -name discovery
```

安装`docker-proxy`协议代理

```text
docker run \
    -ti \
    -d \
    -p 2375:2375 \
    --hostname=$HOSTNAME \
    --restart=always \
    --name shipyard-proxy \
    -v /var/run/docker.sock:/var/run/docker.sock \
    -e PORT=2375 \
    shipyard/docker-proxy:latest
```

安装`Swarm`管理节点

```text
docker run \
    -ti \
    -d \
    --restart=always \
    --name shipyard-swarm-manager \
    swarm:latest \
    manage --host tcp://0.0.0.0:3375 etcd://<IP-OF-HOST>:4001
```

安装`Swarm`从节点

```text
docker run \
    -ti \
    -d \
    --restart=always \
    --name shipyard-swarm-agent \
    swarm:latest \
    join --addr <ip-of-host>:2375 etcd://<ip-of-host>:4001
```

安装`Shipyard`管理界面

```text
docker run \
    -ti \
    -d \
    --restart=always \
    --name shipyard-controller \
    --link shipyard-rethinkdb:rethinkdb \
    --link shipyard-swarm-manager:swarm \
    -p 8080:8080 \
    shipyard/shipyard:latest \
    server \
    -d tcp://swarm:3375
```

访问`http://[ip-of-host]:8080`即可访问web-ui界面

```text
帐号：admin
密码：shipyard
```

增加docker节点

```text
export ACTION=node DISCOVERY=etcd://<ip-of-host>:4001
curl -sSL https://shipyard-project.com/deploy | sh
```

{% hint style="info" %}
安装完以上所有步骤，请重启一次docker服务

systemctl restart docker
{% endhint %}

shipyard 成功界面

![](../../../.gitbook/assets/image%20%2825%29.png)

