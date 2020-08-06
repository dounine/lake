---
description: >-
  为什么这么说呢，难道我自己多年使用的方式错了嘛，是的，你没错，我错了，哈哈，主要原因是使用Tuple的时候容易搞错对象，Tuple2的时候还知道第一个参数跟第二个参数的意思，后面多来个几参数你会记得_1._2._3._4代表的意思是什么吗?代码结构也不好维护，所以请结束使用Tuple吧
---

# Scala比元组还好用的东西

正常使用tuple的方式

```scala
val list = Array((1,2,3,4),(5,6,7,8))
list.filter(_._1>0).map(_._2).foreach(println)
```

{% hint style="info" %}
再过几天你还记得\_1,\_2是什么意思吗，假设list是个变量从其它地方传过来，蛋就更加的疼了，当然了，有小伙伴又说了，我使用case class 不就解决这样的问题了吗？有道理，那如果业务有很多case class 呢？维护起来是不是也很复杂，说了半天，快直接说答案，来了来了，这就一一道来。
{% endhint %}

正确的打开方式

使用匿名类

```scala
new {
        val id:Int
        ...
      }
```

正确例子

```scala
val list = Array(
      new {
        val id: Int = 1
        val age: Int = 2
        val add: Int = 3
        val name: Int = 4
      },
      new {
        val id: Int = 5
        val age: Int = 6
        val add: Int = 7
        val name: Int = 8
      }
    )
    list.filter(_.id>0).map(_.age).foreach(println)
```

