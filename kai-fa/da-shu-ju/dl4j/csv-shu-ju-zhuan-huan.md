# CSV数据转换

准备数据

```text
0,0,24,9.833333333333334,10,9.7,454,0
0,1,4,17.0,1,17.0,432,0
1,0,2,20.0,1,20.0,0,0
1,1,24,10.375,13,9.615384615384615,455,0
1,1,4,10.75,3,11.0,0,0
0,1,3,16.0,2,16.0,246,0
0,1,6,13.0,4,13.0,4767,0
```

转换

```scala
val sparkConf = new SparkConf()
    .set("spark.serializer", "org.apache.spark.serializer.KryoSerializer")
    .set("spark.kryo.registrator", "org.nd4j.Nd4jRegistrator")
    .setMaster("local[*]")
    .setAppName("Dl4jTransform")

  val useSparkLocal = true

  val spark = SparkSession
    .builder
    .config(sparkConf)
    .getOrCreate()

  def main(args: Array[String]): Unit = {
    val sc = spark.sparkContext
    sc.setLogLevel("ERROR")

    val inputDataSchema = new Schema.Builder()
      .addColumnInteger("geneSid")
      .addColumnInteger("platform")
      .addColumnInteger("loginCount")
      .addColumnDouble("loginHour")
      .addColumnInteger("shareCount")
      .addColumnDouble("shareHour")
      .addColumnDouble("regHours")
      .addColumnCategorical("shareIn", "YES", "NO")
      .build()

    val tp = new TransformProcess.Builder(inputDataSchema)
      .removeColumns("shareHour", "loginHour")
      .convertToInteger("regHours") //转成整数
//      .transform(new BaseDoubleTransform("regHours") { //自定义转换
//        override def map(writable: Writable): Writable = {
//          new IntWritable(writable.toInt)
//        }
//
//        override def map(o: Any): AnyRef = {
//          val d = o.asInstanceOf[Double]
//          new IntWritable(d.toInt)
//        }
//      })
      .categoricalToInteger("shareIn") // 转成数字 YES:0  NO:1
      .build()

    val lines = spark.sparkContext.textFile("hello.csv")
    val readWritables = lines.map(new StringToWritablesFunction(new CSVRecordReader()).call(_))
    val processed = SparkTransformExecutor.execute(readWritables, tp)
    val toSave = processed.map(new WritablesToStringFunction("\t"))

    import spark.implicits._
    toSave.rdd.toDS().show(false)
  }
```

输出结果

```scala
+------------------------+
|value                   |
+------------------------+
|0	0	24	10	454	  0  |
|0	1	4	1	432	  0  |
|1	0	2	1	0	  0  |
|1	1	24	13	455	  1  |
|1	1	4	3	0     0  |
|0	1	3	2	246	  0  |
|0	1	6	4	4767  0  |
```

