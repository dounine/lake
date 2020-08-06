---
description: >-
  HDFS_DELEGATION_TOKEN
  这个BUG在很多文章中都出现着，讲了很多原理，但是只给出了官方引用地扯，完全没有给出如何解决，我们线上的业务就有着这样的问题，7天一到马上出现这问题了，官方明明说这个bug修复了呀，因为我们使用的版本是比较新的，理论上不会有这样的问题才对，可是偏偏就有了，没办法，只能硬上了，花了两天的时间找到了解决这个问题的办法，下面会还原这个错误及给出解决方案。
---

# HDFS\_DELEGATION\_TOKEN 还原及解决方案

版本

![](../../../.gitbook/assets/image%20%2825%29.png)

## 配置

添加 hdfs-site.xml 配置

```text
dfs.namenode.delegation.key.update-interval=60000 #1分钟
dfs.namenode.delegation.token.max-lifetime=180000 #3分钟
dfs.namenode.delegation.token.renew-interval=60000 #1分钟
```

修改 /etc/krb5.conf ticket过期为1小时

```text
...
ticket_lifetime = 1h
...
```

测试代码内有`kerberos`认证

{% code title="App.scala" %}
```scala
class App {
  System.setProperty("java.security.krb5.conf", "/etc/krb5.conf")
  System.setProperty("sun.security.krb5.debug", "false")
  val hConf = new Configuration
  hConf.addResource("hbase-site.xml")
  UserGroupInformation.setConfiguration(hConf)
  UserGroupInformation.loginUserFromKeytab("hbase-bd@EXAMPLE.COM", "/etc/security/keytabs/hbase.headless.keytab")

  val sparkConf = new SparkConf()
    //      .setMaster("local[12]")
    .setAppName("HDFS_DELEGATION_TOKEN")
  val spark = SparkSession
    .builder
    .config(sparkConf)
    .getOrCreate()
  hConf.set("hbase.mapreduce.inputtable", "test_log")
  def run(args: Array[String]): Unit = {
    val sc = spark.sparkContext
    import spark.implicits._

    val userRDD: RDD[Log] = sc.newAPIHadoopRDD(
      hConf,
      classOf[TableInputFormat],
      classOf[ImmutableBytesWritable],
      classOf[Result]
    ).flatMap {
      rdd => {
        val map = HbaseUtil.result2Map(rdd._2)
        val log = Log(
          map.get("uid")
        )
        Array(log)
      }
    }

    userRDD.toDS().cache().createTempView("log")

    spark.sql(
      """select * from log""".stripMargin)
      .show(false)

    spark.catalog.dropTempView("log")
    userRDD.unpersist()
  }
}
case class Log(uid: String)
object App {
  def main(args: Array[String]): Unit = {
    val app = new App()
    while (true) {
      app.run(args)
      TimeUnit.MINUTES.sleep(3)
    }
  }
}
```
{% endcode %}

## 测试

测试百度跟谷歌中最最最多出现的解决方案

### 第1种

增加配置

```scala
--conf spark.hadoop.fs.hdfs.impl.disable.cache=true
--conf mapreduce.job.complete.cancel.delegation.tokens=false
```

测试提交

```scala
spark-submit --master yarn \
 --class com.dounine.hbase.App \
 --executor-memory 1g \
 --driver-memory 1g \
 --keytab /etc/security/keytabs/hbase.headless.keytab \
 --principal hbase-bd@EXAMPLE.COM \
 build/libs/hdfs-token-1.0.0-SNAPSHOT-all.jar
```

### 第2种

增加配置

```scala
...
 --conf spark.hadoop.fs.hdfs.impl.disable.cache=true \
 --conf mapreduce.job.complete.cancel.delegation.tokens=false \
...
```

测试提交

```scala
...
 --conf mapreduce.job.complete.cancel.delegation.tokens=false \
...
```

### 第3种

```scala
...
 --conf spark.hadoop.fs.hdfs.impl.disable.cache=true \
...
```

### 结果

```scala
1，2，3，4 测试结果
时间观察3分钟 => **正常**
时间观察10分钟 => **正常**
时间观察30分钟 => **正常**
时间观察60分钟 => **正常**
时间观察120分钟 => **正常**

测试结论 => 与1、2、3、4 --conf 配置无关

好吧，我已经怀疑人生、可能是我打开的方式不对
```

### 继续测试

将认证代码放入run方法内

```scala
def run(args: Array[String]): Unit = {
  System.setProperty("java.security.krb5.conf", "/etc/krb5.conf")
  System.setProperty("sun.security.krb5.debug", "false")
  val hConf = new Configuration
  hConf.addResource("hbase-site.xml")
  UserGroupInformation.setConfiguration(hConf)
  UserGroupInformation.loginUserFromKeytab("hbase-bd@EXAMPLE.COM", "/etc/security/keytabs/hbase.headless.keytab")

  val sparkConf = new SparkConf()
    //      .setMaster("local[12]")
    .setAppName("HDFS_DELEGATION_TOKEN")
  val spark = SparkSession
    .builder
    .config(sparkConf)
    .getOrCreate()

  hConf.set("hbase.mapreduce.inputtable", "test_log")
```

{% hint style="info" %}
时间观察3分钟 =&gt; 正常 

时间观察6分钟 =&gt; 异常
{% endhint %}

```scala
18/12/29 16:50:31 ERROR AsyncEventQueue: Listener EventLoggingListener threw an exception
org.apache.hadoop.ipc.RemoteException(org.apache.hadoop.security.token.SecretManager$InvalidToken): token (token for hbase: HDFS_DELEGATION_TOKEN owner
=hbase-bd@EXAMPLE.COM, renewer=yarn, realUser=, issueDate=1546072104965, maxDate=1546072704965, sequenceNumber=15985, masterKeyId=748) is expired, curr
ent time: 2018-12-29 16:32:29,829+0800 expected renewal time: 2018-12-29 16:31:24,965+0800
        at org.apache.hadoop.ipc.Client.getRpcResponse(Client.java:1497)
        at org.apache.hadoop.ipc.Client.call(Client.java:1443)
        at org.apache.hadoop.ipc.Client.call(Client.java:1353)
        at org.apache.hadoop.ipc.ProtobufRpcEngine$Invoker.invoke(ProtobufRpcEngine.java:228)
        at org.apache.hadoop.ipc.ProtobufRpcEngine$Invoker.invoke(ProtobufRpcEngine.java:116)
        at com.sun.proxy.$Proxy11.fsync(Unknown Source)
        at org.apache.hadoop.hdfs.protocolPB.ClientNamenodeProtocolTranslatorPB.fsync(ClientNamenodeProtocolTranslatorPB.java:980)
        at sun.reflect.GeneratedMethodAccessor11.invoke(Unknown Source)
        at sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43)
        at java.lang.reflect.Method.invoke(Method.java:498)
        at org.apache.hadoop.io.retry.RetryInvocationHandler.invokeMethod(RetryInvocationHandler.java:422)
        at org.apache.hadoop.io.retry.RetryInvocationHandler$Call.invokeMethod(RetryInvocationHandler.java:165)
        at org.apache.hadoop.io.retry.RetryInvocationHandler$Call.invoke(RetryInvocationHandler.java:157)
        at org.apache.hadoop.io.retry.RetryInvocationHandler$Call.invokeOnce(RetryInvocationHandler.java:95)
        at org.apache.hadoop.io.retry.RetryInvocationHandler.invoke(RetryInvocationHandler.java:359)
        at com.sun.proxy.$Proxy12.fsync(Unknown Source)
```

## 问题发现

通过几十遍不断的调整位置、最终确认的问题所在是Exector的问题

```scala
UserGroupInformation.setConfiguration(hConf)
UserGroupInformation.loginUserFromKeytab("hbase-bd@EXAMPLE.COM", "/etc/security/keytabs/hbase.headless.keytab")
```

是由于以上两句kerberos认证代码导致的结果

跟下面的配置冲突了

```scala
--principal $principal --keytab $keytab
```

## 解决方案\(一\)

删除掉下面代码中的这两句认证即可,使用`--principal $principal --keytab $keytab`

```scala
UserGroupInformation.setConfiguration(hConf)
UserGroupInformation.loginUserFromKeytab("hbase-bd@EXAMPLE.COM", "/etc/security/keytabs/hbase.headless.keytab")
```

{% hint style="info" %}
因为Spark的`--principal --keytab`会在令牌即将过期的时候帮我们重新续定，如果代码里面加上之后，Spark会读取到ApplicationMaster中用户已经认证了，没有过期是不会续定NodeManager中的Exector的。 如果是开发环境模式，可以加一个判断使用以上两句代码 **简单粗暴**
{% endhint %}

## 解决方案\(二\)

使用UserGroupInformation的进程认证方式

```scala
spark.sparkContext
      .parallelize(0 to 1000)
      .repartition(10)
      .foreachPartition {
        iter => {
          val hConf = new Configuration
          hConf.addResource("hbase-site.xml")
          val ugi = UserGroupInformation.loginUserFromKeytabAndReturnUGI("hbase-bd@EXAMPLE.COM", "/etc/security/keytabs/hbase.headless.keytab")
          ugi.doAs(new PrivilegedAction[Unit] {//在每个Partition认证
            override def run(): Unit = {
              val logDir = new Path(args(0))
              val fs = FileSystem.get(hConf)
              if (!fs.exists(logDir)) throw new Exception(logDir.toUri.getPath + " director not exist.")

              while (iter.hasNext) {
                iter.next()
                val logPaths = fs.listFiles(logDir, false)
                TimeUnit.MILLISECONDS.sleep(10)
              }
            }
          })
        }
      }
```

## BUG 7天后再次复现

上面推导还是有问题，还有望知道BUG解决的小伙伴告知一下。

## 最终解决方案

就是加大过期的时间











