---
description: >-
  有时候有没有这么一种情况，我拿到了一个sql,csv,parquet文件，一起来就想写sql，不想写那些乱七八糟的的东西，只是想快速实现我要的聚合查询数据。那么我们可以利用spark-sql直接操作文件的特性处理这类的需求，姐姐再也不用担心我不会spark了，因为我就只会sql。
---

# Spark SQL查询文件数据

## 使用

csv 文件

```text
spark.sql("select * from csv.`/tmp/demo.csv`").show(false)
```

json 文件

```text
spark.sql("select * from json.`/tmp/demo.json`").show(false)
```

parquet 文件

```text
spark.sql("select * from parquet.`/tmp/demo.parquet`").show(false)
```



