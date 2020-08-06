---
description: >-
  有没有这样一样情况，把一个集群中的某个表导到另一个群集中，或者hbase的表结构发生了更改，但是数据还要，比如预分区没做，导致某台RegionServer很吃紧，Hbase的导出导出都可以很快的完成这些操作
---

# 迁移数据

## 环境依赖

现在环境上面有一张表`logTable`，有一个`ext`列簇 但是没有做预分区，虽然可以强制拆分表，但是split的start,end范围无法精确控制

## 方式一

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





