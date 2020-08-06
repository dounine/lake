---
description: >-
  在电商领域通常会有这样一种需要，如果客户下单了，但是在10分钟内不付款，应该需要通知客服，再由客服寻问客户为什么还没有付款，从而提高付款效率，我们可以采用Flink
  - CEP的超时机制来处理。
---

# CEP监控下单付款超时通知

执行流程

![](../../../.gitbook/assets/image%20%2838%29.png)

添加依赖

```text
compile group: 'org.apache.flink', name: 'flink-streaming-scala_2.11', version: "1.6.2"
compile group: 'org.apache.flink', name: 'flink-cep-scala_2.11', version: "1.6.2"
```

使用Iterator模拟用户下单

```scala
case class OrderEvent(
                       userId: String,
                       `type`: String
                     )
class DataSource extends Iterator[OrderEvent] with Serializable {
  val atomicInteger = new AtomicInteger(0)

  val orderEventList = List(
    OrderEvent("1", "create"),
    OrderEvent("2", "create"),
    OrderEvent("2", "pay")
  )

  override def hasNext: Boolean = {
    TimeUnit.SECONDS.sleep(1)
    true
  }

  override def next(): OrderEvent = {
    orderEventList(atomicInteger.getAndIncrement() % 3)
  }
}
```

创建定单流

```scala
val orderEventStream = env.fromCollection(new DataSource())
```

创建定单匹配流程

> 以下表示如果在1秒钟内创建定单并付款则完成购物操作

```scala
val orderPayPattern = Pattern.begin[OrderEvent]("begin")
      .where(_.`type`.equals("create"))
      .next("next")
      .where(_.`type`.equals("pay"))
      .within(Time.seconds(1))
```

创建侧输出流

```scala
val orderTiemoutOutput = OutputTag[OrderEvent]("orderTimeout")
```

把定单流应用到匹配流程中

```scala
val patternStream = CEP.pattern(orderEventStream.keyBy("userId"), orderPayPattern)
```

将正常定单流与侧超时流分开

```scala
val complexResult = patternStream.select(orderTiemoutOutput) {
      (pattern: Map[String, Iterable[OrderEvent]], timestamp: Long) => {
        val createOrder = pattern.get("begin")
        OrderEvent("timeout", createOrder.get.iterator.next().userId)
      }
    } {
      pattern: Map[String, Iterable[OrderEvent]] => {
        val payOrder = pattern.get("next")
        OrderEvent("success", payOrder.get.iterator.next().userId)
      }
    }
```

将正常定单流与超时定单流打印输出

```scala
val timeoutResult = complexResult.getSideOutput(orderTiemoutOutput)

complexResult.print()
timeoutResult.print()

env.execute
```

也可以自行添加Sink将消息发送到消息队列等

```scala
timeoutResult.addSink(new SinkFunction[OrderEvent] {
      override def invoke(value: OrderEvent, context: SinkFunction.Context[_]): Unit = {
        //do something or send message
      }
    })
```

[项目源码](https://github.com/dounine/flink-cep-demos/tree/master/order-timeout)

