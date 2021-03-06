---
description: >-
  有没有这样一样情况，把一个集群中的某个表导到另一个群集中，或者hbase的表结构发生了更改，但是数据还要，比如预分区没做，导致某台RegionServer很吃紧，Hbase的导出导出都可以很快的完成这些操作
---

# 迁移数据

## 环境依赖

现在环境上面有一张表`logTable`，有一个`ext`列簇 但是没有做预分区，虽然可以强制拆分表，但是split的start,end范围无法精确控制

## 使用步骤

创建导出目录

```bash
hadoop fs -mkdir /tmp/hbase-export
```

备份表数据

```bash
hbase org.apache.hadoop.hbase.mapreduce.Export \
 -D hbase.mapreduce.scan.column.family=ext \ 
logTable hdfs:///tmp/hbase-export/logTable
```

删除表

```bash
disable 'logTable'
drop 'logTable'
```

创建表

> 数据3天过期，可以根据`RegionServer`的个数做预分区，假设有8台，则使用下面的方式。 由于我们是使用`MD5("uid")`前两位作为打散,范围为`00~ff` 256个分片,可以使用如下方式。

scala shell 创建方式

```bash
(0 until 256 by 256/8).map(Integer.toHexString).map(i=>s"0$i".takeRight(2))
```

hbase创表

```bash
create 'logTable',{ \
  NAME => 'ext',TTL => '3 DAYS', \
  CONFIGURATION => {
  'SPLIT_POLICY' => 'org.apache.hadoop.hbase.regionserver.KeyPrefixRegionSplitPolicy', \
  'KeyPrefixRegionSplitPolicy.prefix_length' => '2' 
  }, \
  COMPRESSION => 'SNAPPY' \
}, \ 
SPLITS => ['20', '40', '60', '80', 'a0', 'c0', 'e0']
```

导出备份数据

```bash
hbase org.apache.hadoop.hbase.mapreduce.Import logTable hdfs:///tmp/hbase-export/logTable
```

{% hint style="info" %}
默认导出所有列簇，可通过指定`-D hbase.mapreduce.scan.column.family=ext,info`参数导出。
{% endhint %}

{% hint style="info" %}
如果另一张表没有源表对应的列簇将会出错。
{% endhint %}

