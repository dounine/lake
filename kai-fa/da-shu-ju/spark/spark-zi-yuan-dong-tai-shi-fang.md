---
description: >-
  通过spark-submit会固定占用一占的资源，有什么办法，在任务不运作的时候将资源释放，让其它任务使用呢，yarn新版本默认已经支持了，这里使用的是HDP演示。
---

# Spark 资源动态释放

版本

![](../../../.gitbook/assets/image%20%287%29.png)

HDP里面已经默认支持spark动态资源释配置

代码配置

```scala
val sparkConf = new SparkConf()
    .set("spark.shuffle.service.enabled", "true")
    .set("spark.dynamicAllocation.enabled", "true")
    .set("spark.dynamicAllocation.minExecutors", "1") //最少占用1个Executor
    .set("spark.dynamicAllocation.initialExecutors", "1") //默认初始化一个Executor
    .set("spark.dynamicAllocation.maxExecutors", "6") //最多占用6个Executor
    .set("spark.dynamicAllocation.executorIdleTimeout", "60") //executor闲置时间
    .set("spark.dynamicAllocation.cachedExecutorIdleTimeout", "60") //cache闲置时间
    .set("spark.executor.cores", "3")//使用的vcore
    //    .setMaster("local[12]")
    .setAppName("Spark DynamicRelease")

  val spark: SparkSession = SparkSession
    .builder
    .config(sparkConf)
    .getOrCreate()
```

{% hint style="info" %}
如果spark计算当中使用了rdd.cache，不加下面的配置，动态资源不会释放

.set\("spark.dynamicAllocation.cachedExecutorIdleTimeout", "60"\)
{% endhint %}



