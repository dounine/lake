---
description: >-
  在spark的数据源中，只支持Append, Overwrite, ErrorIfExists,
  Ignore,这几种模式，但是我们在线上的业务几乎全是需要upsert功能的，就是已存在的数据肯定不能覆盖，在mysql中实现就是采用：`ON
  DUPLICATE KEY
  UPDATE`，有没有这样一种实现？官方：不好意思，不提供，dounine：我这有呀，你来用吧。哈哈，为了方便大家的使用我已经把项
---

# Spark 升级版数据源JDBC2

吃土的方案

{% code title="MysqlClient.scala" %}
```scala
import java.sql._
import java.time.{LocalDate, LocalDateTime}
import scala.collection.mutable.ListBuffer
class MysqlClient(jdbcUrl: String) {
  private var connection: Connection = null
  val driver = "com.mysql.jdbc.Driver"
  init()
  def init(): Unit = {
    if (connection == null || connection.isClosed) {
      val split = jdbcUrl.split("\\|")
      Class.forName(driver)
      connection = DriverManager.getConnection(split(0), split(1), split(2))
    }
  }
  def close(): Unit = {
    connection.close()
  }
  def execute(sql: String, params: Any*): Unit = {
    try {
      val statement = connection.prepareStatement(sql)
      this.fillStatement(statement, params: _*)
      statement.executeUpdate
    } catch {
      case e: SQLException =>
        e.printStackTrace()
    }
  }
  @throws[SQLException]
  def fillStatement(statement: PreparedStatement, params: Any*): Unit = {
    for (i <- 1 until params.length + 1) {
      val value: Any = params(i - 1)
      value match {
        case s: String => statement.setString(i, value.toString)
        case i: Integer => statement.setInt(i, value.toString.asInstanceOf[Int])
        case b: Boolean => statement.setBoolean(i, value.toString.asInstanceOf[Boolean])
        case ld: LocalDate => statement.setString(i, value.toString)
        case ldt: LocalDateTime => statement.setString(i, value.toString)
        case l: Long => statement.setLong(i, value.toString.asInstanceOf[Long])
        case d: Double => statement.setDouble(i, value.toString.asInstanceOf[Double])
        case f: Float => statement.setFloat(i, value.toString.asInstanceOf[Float])
        case _ => statement.setString(i, value.toString)
      }
    }
  }
  def upsert(query: Query, update: Update, tableName: String): Unit = {
    val names = ListBuffer[String]()
    val values = ListBuffer[String]()
    val params = ListBuffer[AnyRef]()
    val updates = ListBuffer[AnyRef]()
    val keysArr = scala.Array(query.values.keys, update.sets.keys, update.incs.keys)
    val valuesArr = scala.Array(update.sets.values, update.incs.values)
    for (i: Int <- 0 until keysArr.length) {
      val item = keysArr(i)
      item.foreach {
        key => {
          names += s"`${key}`"
          values += "?"
        }
      }
      i match {
        case 0 => {
          params.++=(query.values.values)
        }
        case 1 | 2 => {
          params.++=(valuesArr(i - 1).toList)
        }
      }
    }
    update.sets.foreach {
      item => {
        updates += s" `${item._1}` = ? "
        params += item._2
      }
    }
    update.incs.foreach {
      item => {
        updates += s" `${item._1}` = `${item._1}` + ? "
        params += item._2
      }
    }
    val sql = s"INSERT INTO `$tableName` (${names.mkString(",")}) VALUES(${values.mkString(",")}) ON DUPLICATE KEY UPDATE ${updates.mkString(",")}"
    this.execute(sql, params.toArray[AnyRef]: _*)
  }
}
case class Update(sets: Map[String, AnyRef] = Map(), incs: Map[String, AnyRef] = Map())
case class Query(values: Map[String, AnyRef] = Map())
```
{% endcode %}

入口程序

```scala
val fieldMaps = (row: Row, fields: Array[String]) => fields.map {
    field => (field, Option(row.getAs[String](field)).getOrElse(""))
  }.toMap
sc.sql(
      s"""select time,count(userid) as pv,count(distinct(userid)) as uv from log group by time""")
      .foreachPartition(item => {
        val props: Properties = PropertiesUtils.properties("mysql")
        val mysqlClient: MysqlClient = new MysqlClient(props.getProperty("jdbcUrl"))
        while (item.hasNext) {
          val row: Row = item.next()
          val pv: Long = row.getAs("pv")
          val uv: Long = row.getAs("uv")
          val indicatorMap = Map(
           "pv" -> pv.toString,
           "uv" -> uv.toString
          )

          val update = if (overrideIndicator) {//覆盖
            Update(sets = indicatorMap)
          } else {//upsert
            Update(incs = indicatorMap)
          }

          var queryMap = fieldMaps(row,"time".split(","))

          mysqlClient.upsert(
            Query(queryMap),
            update,
            "test"
          )

        }
        mysqlClient.close()
      })
```

{% hint style="info" %}
真的是丑极了，不想看了
{% endhint %}

## 升级后

依赖 [spark-sql-datasource](https://mvnrepository.com/artifact/com.dounine/spark-sql-datasource)

```scala
<dependency>
  <groupId>com.dounine</groupId>
  <artifactId>spark-sql-datasource</artifactId>
  <version>1.0.1</version>
</dependency>
```

创建一张测试表

```scala
CREATE TABLE `test` (
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `time` date NOT NULL,
  `pv` int(255) DEFAULT '0',
  `uv` int(255) DEFAULT '0',
  PRIMARY KEY (`id`),
  UNIQUE KEY `uniq` (`time`) USING BTREE
) ENGINE=InnoDB AUTO_INCREMENT=22 DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_bin;
```

测试

```scala
val spark = SparkSession
      .builder()
      .appName("jdbc2")
      .master("local[*]")
      .getOrCreate()

    val readSchmeas = StructType(
      Array(
        StructField("userid", StringType, nullable = false),
        StructField("time", StringType, nullable = false),
        StructField("indicator", LongType, nullable = false)
      )
    )

    val rdd = spark.sparkContext.parallelize(
      Array(
        Row.fromSeq(Seq("lake", "2019-02-01", 10L)),
        Row.fromSeq(Seq("admin", "2019-02-01", 10L)),
        Row.fromSeq(Seq("admin", "2019-02-01", 11L))
      )
    )

    spark.createDataFrame(rdd, readSchmeas).createTempView("log")

    spark.sql("select time,count(userid) as pv,count(distinct(userid)) as uv from log group by time")
      .write
      .format("org.apache.spark.sql.execution.datasources.jdbc2")
      .options(
        Map(
          "savemode" -> JDBCSaveMode.Update.toString,
          "driver" -> "com.mysql.jdbc.Driver",
          "url" -> "jdbc:mysql://localhost:3306/ttable",
          "user" -> "root",
          "password" -> "root",
          "dbtable" -> "test",
          "useSSL" -> "false",
          "duplicateIncs" -> "pv,uv",
          "showSql" -> "true"
        )
      ).save()
```

{% hint style="info" %}
实际程序上运行会生成下面的 SQL 语句

INSERT INTO test \(`time`,`pv`,`uv`\) VALUES \(?,?,?\) ON DUPLICATE KEY UPDATE `time`=?,`pv`=`pv`+?,`uv`=`uv`+?
{% endhint %}



