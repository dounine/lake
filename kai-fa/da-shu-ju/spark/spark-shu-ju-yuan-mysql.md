---
description: >-
  如果我们的Mysql服务器性能不咋滴，但是硬盘很够，如何才能做各种复杂的聚合操作？答案就是使用spark的计算能力的，我们可以将mysql数据源接入到spark中。
---

# Spark 数据源MySQL

## 查询

测试

```scala
val mysqlDF = spark
  .read
  .format("jdbc")
  .option("driver","com.mysql.jdbc.Driver")
  .option("url","jdbc:mysql://localhost:3306/ttable")
  .option("user","root")
  .option("password","root")
  .option("dbtable","(select * from ttt where userId >1 AND userId < 10) as log")//条件查询出想要的表
  //.option("dbtable","ttable.ttt")//整张表
  .option("fetchsize","100")
  .option("useSSL","false")
  .load()
```

分区读取

```scala
spark
  .read
  .format("jdbc")
  .option("url", url)
  .option("dbtable", "ttt")
  .option("user", user)
  .option("password", password)
  .option("numPartitions", 10)
  .option("partitionColumn", "userId")
  .option("lowerBound", 1)
  .option("upperBound", 10000)
  .load()
```

实际会生成如下查询语句,\(所有分区会一直查询，直到整张表数据查询完为止\)

```scala
SELECT * FROM ttt WHERE userId >= 1 and userId < 1000
SELECT * FROM ttt WHERE userId >= 1000 and userId < 2000
SELECT * FROM ttt WHERE userId >= 2000 and userId < 3000
```

## 写入

```scala
mysqlDF.createTempView("log")

spark
  .sql("select * from log")
  .toDF()
  .write
  .mode(SaveMode.Overwrite)
  .format("jdbc")
  .option("driver","com.mysql.jdbc.Driver")
  .option("url","jdbc:mysql://localhost:3306/ttable")
  .option("dbtable","a")
  .option("user","root")
  .option("password","root")
  .option("fetchsize","100")
  .option("useSSL","false")
  .save()
```



