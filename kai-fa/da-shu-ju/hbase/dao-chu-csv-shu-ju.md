---
description: 使用它导出了3亿的用户数据出来，花了半个小时，性能还是稳稳的，好了不吹牛皮了，直接上代码吧。
---

# 导出CSV数据

导出的CSV格式为

```text
admin,22,北京
admin,23,天津
```

添加依赖包

```text
<dependency>
    <groupId>org.apache.hbase</groupId>
    <artifactId>hbase-mapreduce</artifactId>
    <version>2.3.0</version>
</dependency>
```

定义Map转换类

```scala
class MyMapper extends TableMapper[Text, Text] {

  val keyText = new Text()
  val valueText = new Text()

  override def map(key: ImmutableBytesWritable, value: Result, context: Mapper[ImmutableBytesWritable, Result, Text, Text]#Context): Unit = {
    val maps = result2Map(value)
    keyText.set(maps.get("userId"))
    valueText.set(s"${maps.get("regTime")}")
    context.write(keyText, valueText)
  }

  //将Result转换为Map
  def result2Map(result: Result): util.HashMap[lang.String, lang.String] = {
    val map = new util.HashMap[lang.String, lang.String]()
    result.rawCells().foreach {
      cell =>
        val column: Array[Byte] = CellUtil.cloneQualifier(cell)
        val value: Array[Byte] = CellUtil.cloneValue(cell)
        val qualifierByte = cell.getQualifierArray
        if (qualifierByte != null && qualifierByte.nonEmpty) {
          if (value == null || value.length == 0) {
            map.put(Bytes.toString(column), "")
          } else {
            map.put(Bytes.toString(column), Bytes.toString(value))
          }
        }
    }
    map
  }

}
```

定义Reducer类

```scala
class MyReducer extends Reducer[Text, Text, Text, Text] {
  override def reduce(key: Text, values: lang.Iterable[Text], context: Reducer[Text, Text, Text, Text]#Context): Unit = {
    val iter = values.iterator()
    while (iter.hasNext) {
     //这样可以只保留下Key字段，也就只有一行数据了
      val tmpText = iter.next()
      val mergeKey = new Text()
      mergeKey.set(key.toString + "," + tmpText.toString)
      val v = new Text()
      v.set("")
      context.write(mergeKey, v)
    }
  }
}
```

ExportCsv 执行入口

```scala
class ExportCsv extends Configured with Tool {

  override def run(args: Array[String]): Int = {
    val conf = HBaseConfiguration.create()
    conf.addResource(new FileInputStream(new File("/etc/hbase/conf/hbase-site.xml")))
    conf.set(org.apache.hadoop.mapreduce.lib.output.FileOutputFormat.OUTDIR, "/tmp/hbasecsv")
    conf.set("mapreduce.job.running.map.limit", "8") //最多有多少个Task同时跑

    val job = Job.getInstance(conf, "HbaseExportCsv")
    job.setJarByClass(classOf[ExportCsv])

    val scan = new Scan()

    //过滤我们想要的数据
    scan.addFamily(Bytes.toBytes("ext"))
    scan.addColumn(Bytes.toBytes("ext"), Bytes.toBytes("userId"))
    scan.addColumn(Bytes.toBytes("ext"), Bytes.toBytes("regTime"))

    scan.setBatch(1000)
    scan.setCacheBlocks(false)

    TableMapReduceUtil.initTableMapperJob(
      "USER_TABLE",
      scan,
      classOf[MyMapper],
      classOf[Text],
      classOf[Text],
      job
    )
    job.setReducerClass(classOf[MyReducer])
    val jobConf = new JobConf(job.getConfiguration)
    FileOutputFormat.setOutputPath(jobConf, new Path("/tmp/hbasecsv"))
    val isDone = job.waitForCompletion(true)
    if (isDone) 0 else 1
  }
}
```

运行任务

```scala
hadoop jar ExportCsv.jar
```

