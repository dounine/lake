---
description: >-
  这里有17个G的日志文件，使用多线程压缩2分23秒即可压缩完成3.2G的压缩，6倍的压缩比，普通压缩则要使用7分50秒，整整多出了3倍，我们看看是怎么使用的。
---

# 多线程压缩日志

安装pigz

```text
yum install pigz -y
# 或者
apt-get install pigz
```

压缩\(使用8个CPU\)

```text
time tar -cf - logdir | pigz -p 8 > logdir.tgz
```

解压缩

```text
time pigz -p 8 -d logdir.tgz
```

{% hint style="info" %}
速度是非常之快的，大家可以自行测试。
{% endhint %}

