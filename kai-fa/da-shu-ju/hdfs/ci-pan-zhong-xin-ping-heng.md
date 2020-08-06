---
description: >-
  在日积月累的操作中、可能会存在有些磁盘的存储分布得不是很平衡、这就给数据多的那一台机子带来压力、因为很多的读取都是在同一台机子上、所以我们需要重新平衡一下存储、也就是把存储多的机子上的数据转移到其它机子。这里我们使用hdfs提供的balancer命令操作
---

# 磁盘重新平衡

随意登录hdfs集群中的某一台机子、然后切换到hdfs用户

```text
$ su - hdfs
```

kerberos 认证\[可选\]

```text
$ kinit -kt /etc/security/keytabs/hdfs.headless.keytab hdfs-demo
```

平衡命令

```text
$ hdfs balancer -threshold 5
```

