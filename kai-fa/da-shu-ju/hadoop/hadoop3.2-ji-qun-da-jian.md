---
description: >-
  Hadoop3.2
  集群新版本的搭建详细讲解过程，从下面第一张官方的图来看，最新版是3.2，所以大猪将使用3.2的版本来演示，过程中遇到的坑留给自己，把路留给你们
---

# Hadoop3.2 集群搭建

版本选择

![](../../../.gitbook/assets/image%20%2822%29.png)

## 环境

下面所有操作都使用root帐号

> 准备两台主机 10.211.55.11 、 10.211.55.12
>
> 对应的hostname为 m1.example.com 、 m2.example.com

## 配置

hostname 设置

![](../../../.gitbook/assets/image%20%2837%29.png)

主机免密钥登录



![](../../../.gitbook/assets/image%20%2825%29.png)

同步hosts

![](../../../.gitbook/assets/image%20%2811%29.png)

下载 [jdk1.8](https://www.oracle.com/technetwork/java/javase/downloads/jdk8-downloads-2133151.html)

![](../../../.gitbook/assets/image%20%2839%29.png)

配置Java环境

![](../../../.gitbook/assets/image%20%2832%29.png)

环境变量生效

```text
source /etc/profile
```

两台主机创建好hadoop的文件夹

![](../../../.gitbook/assets/image%20%2823%29.png)

环境配置 [hadoop-3.2.0.tar.gz](https://www.apache.org/dyn/closer.cgi/hadoop/common/hadoop-3.2.0/hadoop-3.2.0.tar.gz)

![](../../../.gitbook/assets/image%20%2829%29.png)

### 参数配置

运行用户操作，添加如下几句到 `/soft/hadoop/etc/hadoop/hadoop-env.sh` 文件

![](../../../.gitbook/assets/image%20%2818%29.png)

### hadoop 配置文件

core-site.xml 配置

![](../../../.gitbook/assets/image%20%2821%29.png)

hdfs-site.xml 配置

![](../../../.gitbook/assets/image%20%2820%29.png)

yarn-site.xml 配置

![](../../../.gitbook/assets/image%20%2830%29.png)

mapred-site.xml 配置

![](../../../.gitbook/assets/image%20%2840%29.png)

格式化name文件夹

```text
hadoop namenode -format
```

启动

```text
start-all.sh
```

![](../../../.gitbook/assets/image%20%2831%29.png)

访问 [http://m1.example.com:8088/cluster](http://m1.example.com:8088/cluster)

![](../../../.gitbook/assets/image%20%283%29.png)

