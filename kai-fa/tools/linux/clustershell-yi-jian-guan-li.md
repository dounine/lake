---
description: >-
  在集群的现在，管理成千上万的`Linux`机子成为了一个难题，不能再使用一台一台登录主机去进行管理了，工作量无疑是非常巨大的，这里我们可以借助`Clustershell`与`pssh`管理，它原理是自动登录主机集群帮我们将脚本进行执行，或者是文件的操作等，当初大猪是一台一台去管理的，一个字：好累呀。
---

# Clustershell 一键管理

## 环境

Centos7

## 安装

我们只需要在一台主机上操作即可完成对集群机子的所有配置

```text
master      host.com      #主机
cluster1    2.host.com   	#节点1
cluster2    3.host.com  	#节点2
cluster3    4.host.com  	#节点3
```

### 免密码登录

主机使用ssh登录1,2,3节点必须是**免密码登录方式**，并且不能是首次登录，因为`Clustershell`不会自动输入交互命令，例如第一次登录主机会有下面提示。

```text
ssh root@2.host.com

The authenticity of host '2.host.com (192.168.0.20)' can't be established.
ECDSA key fingerprint is d4:5d:d6:e6:bc:70:86:1b:42:32:aa:6b:86:a6:34:d4.
Are you sure you want to continue connecting (yes/no)?
```

`Clustershell`不能自动处理，会提示以下错误

```text
[1] 11:32:17 [FAILURE] 2.host.com Exited with error code 255
```

生成密钥

```text
ssh-keygen -t rsa
```

公钥内容复制到各个节点上

```text
scp ~/.ssh/id_rsa.pub 192.168.0.20:/root/.ssh/authorized_keys
scp ~/.ssh/id_rsa.pub 192.168.0.21:/root/.ssh/authorized_keys
scp ~/.ssh/id_rsa.pub 192.168.0.22:/root/.ssh/authorized_keys
```



## 配置

{% code title="/etc/clustershell/groups.d/local.cfg" %}
```text
# ClusterShell groups config local.cfg
#
# Replace /etc/clustershell/groups
#
# Note: file auto-loaded unless /etc/clustershell/groups is present
#
# See also groups.d/cluster.yaml.example for an example of multiple
# sources single flat file setup using YAML syntax.
#
# Feel free to edit to fit your needs.
adm: example0
oss: example4 example5
mds: example6
io: example[4-6]
compute: example[32-159]
gpu: example[156-159]
all: example[4-6,32-159]
demo: [1-3].host.com
```
{% endcode %}

以上我们定义了一个分组：demo，通过这个分组可连接1.host.com,2.host.com,3.host.com，如果此三个节点对应的IP地扯没有映射有连接失败，可设置/etc/hosts对应关系

{% code title="/etc/hosts" %}
```text
127.0.0.1 localhost
::1         localhost localhost.localdomain localhost6 localhost6.localdomain6

192.168.0.20 1.host.com
192.168.0.21 2.host.com
192.168.0.22 3.host.com
```
{% endcode %}

clush命令几个重要的参数

```text
-b : 相同输出结果合并
-w : 指定节点
-a : 所有节点
-g : 指定组
--copy : 群发文件
```

各个节点执行`ls`命令

```text
clush -g demo "ls"
```

创建文件

```text
clush -g demo "touch /root/demo.txt"
```

群发文件

```text
clush -g demo --copy groups --dest /root  
```





