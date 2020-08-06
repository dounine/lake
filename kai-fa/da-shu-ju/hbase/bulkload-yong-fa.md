---
description: >-
  要导入大量数据，Hbase的BulkLoad是必不可少的，在导入历史数据的时候，我们一般会选择使用BulkLoad方式，我们还可以借助Spark的计算能力将数据快速地导入。
---

# BulkLoad用法

添加依赖包

```text
compile group: 'org.apache.spark', name: 'spark-sql_2.11', version: '2.3.1.3.0.0.0-1634'
compile group: 'org.apache.spark', name: 'spark-core_2.11', version: '2.0.0.3.0.0.0-1634'
compile group: 'org.apache.hbase', name: 'hbase-it', version: '2.0.0.3.0.0.0-1634'
```

创建好表

```text
create 'test_log','ext'
```

BulkLoad.scala 核心代码

```scala
def main(args: Array[String]): Unit = {
    val sparkConf = new SparkConf()
      //      .setMaster("local[12]")
      .setAppName("HbaseBulkLoad")

    val spark = SparkSession
      .builder
      .config(sparkConf)
      .getOrCreate()
    val sc = spark.sparkContext

    val datas = List(//模拟200亿数据
      ("abc", ("ext", "type", "login")),
      ("ccc", ("ext", "type", "logout"))
    )
    val dataRdd = sc.parallelize(datas)

    val output = dataRdd.map {
      x => {
        val rowKey = Bytes.toBytes(x._1)
        val immutableRowKey = new ImmutableBytesWritable(rowKey)

        val colFam = x._2._1
        val colName = x._2._2
        val colValue = x._2._3

        val kv = new KeyValue(
          rowKey,
          Bytes.toBytes(colFam),
          Bytes.toBytes(colName),
          Bytes.toBytes(colValue.toString)
        )
        (immutableRowKey, kv)
      }
    }


    val hConf = HBaseConfiguration.create()
    hConf.addResource("hbase-site.xml")
    val hTableName = "test_log"
    hConf.set("hbase.mapreduce.hfileoutputformat.table.name", hTableName)
    val tableName = TableName.valueOf(hTableName)
    val conn = ConnectionFactory.createConnection(hConf)
    val table = conn.getTable(tableName)
    val regionLocator = conn.getRegionLocator(tableName)

    val hFileOutput = "/tmp/h_file"

    output.saveAsNewAPIHadoopFile(hFileOutput,
      classOf[ImmutableBytesWritable],
      classOf[KeyValue],
      classOf[HFileOutputFormat2],
      hConf
    )

    val bulkLoader = new LoadIncrementalHFiles(hConf)
    bulkLoader.doBulkLoad(new Path(hFileOutput), conn.getAdmin, table, regionLocator)
  }
```

提交Spark任务

```scala
spark-submit --master yarn --conf spark.yarn.tokens.hbase.enabled=true --class com.dounine.hbase.BulkLoad --executor-memory 2G --num-executors 2G --driver-memory 2G    --executor-cores 2 build/libs/hbase-data-insert-1.0.0-SNAPSHOT-all.jar
```

项目源码：[https://github.com/dounine/hbase-data-insert](https://github.com/dounine/hbase-data-insert)

