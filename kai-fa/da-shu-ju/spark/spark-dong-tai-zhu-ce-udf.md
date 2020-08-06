---
description: >-
  昨天有位大哥问小弟一个Spark问题，他们想在不停Spark程序的情况下动态更新UDF的逻辑，他一问我这个问题的时候，本猪心里一惊，Spark**还能这么玩?我出于程序员的本能回复他肯定不行，但今天再回过来头想了一想，昨天脑子肯定进水了，回复太肤浅了，既然Spark可以通过编程方式注册UDF，当然把那位大哥的代码逻辑使用反射加载进去再调用不就行了？这不就是JVM的优势么，怪自己的反射没学到家，说搞
---

# Spark 动态注册UDF

## 分析过程

看一个`Spark`注册`UDF`的例子

```scala
spark.udf.register(name, (a1: String) => a1.toUpperCase)
```

点击`register`的源码进去看

```scala
一个`A1`:参数类型,`RT`:返回类型
def register[RT: TypeTag, A1: TypeTag](name: String, func: Function1[A1, RT]): UserDefinedFunction = {
    val ScalaReflection.Schema(dataType, nullable) = ScalaReflection.schemaFor[RT]
    val inputTypes = Try(ScalaReflection.schemaFor[A1].dataType :: Nil).toOption
    def builder(e: Seq[Expression]) = if (e.length == 1) {
      ScalaUDF(func, dataType, e, inputTypes.getOrElse(Nil), Some(name), nullable, udfDeterministic = true)
    } else {
      ...
    }
    ...
  }
def register[RT: TypeTag, A1: TypeTag, A2: TypeTag](name: String, func: Function2[A1, A2, RT]): UserDefinedFunction
def register[RT: TypeTag, A1: TypeTag, A2: TypeTag, A3: TypeTag](name: String, func: Function3[A1, A2, A3, RT]): UserDefinedFunction = {
def register[RT: TypeTag, A1: TypeTag, A2: TypeTag, A3: TypeTag, A4: TypeTag](name: String, func: Function4[A1, A2, A3, A4, RT]): UserDefinedFunction = {
```

1. func是上面方法的重点，既然想要动态`UDF`逻辑代码，那我们把`Function1`这个函数实现不就可以了？再利用JVM反射的技术调用，完美。
2. 顺便还看出了在scala-2.10.x版本中`case class`的元素是不能超过 22 个的。

上面的`UDF`注册的原型其实是

```scala
val udf = new Function1[String,String] {
      override def apply(a1: String): String = {
        a1.toUpperCase
      }
}
spark.udf.register(name, udf)
```

到这里我有一个 **肤浅** 并且 **大胆** 的想法，我把那位大哥的代码放到apply方法里面调用不就行了？

```scala
val udf = new Function1[String,String] {
      override def apply(a1: String): String = {
        //method.invoke(instance) //使用反射加载代码，把大哥动态逻辑方法method拿出来调用。
      }
}
```

1. 但是还有一些问题要解决，我不能强制我的老大哥只能传递一个参数吧，那也太年轻不懂事了，至少让他可以随意传 22 参数。
2. 唯一的解决方法，就是要控制`Function1`到`Function22`函数的动态生成，找了半天没发现`Function`的动态生成，然后还发现Spark也是根据参数长度生成个`FunctionN 方法`
3. 既然实现方式找到了，那就简单了，只要通过反射就能 上知天文,下知地理 。

既然是`Spark`，肯定要用`Scala`去写反射了。

```scala
case class ClassInfo(clazz: Class[_], instance: Any, defaultMethod: Method, methods: Map[String, Method], func:String) {
  def invoke[T](args: Object*): T = {
    defaultMethod.invoke(instance, args: _*).asInstanceOf[T]
  }
}
object ClassCreateUtils extends Logging{
  private val clazzs = new util.HashMap[String, ClassInfo]()
  private val classLoader = scala.reflect.runtime.universe.getClass.getClassLoader
  private val toolBox = universe.runtimeMirror(classLoader).mkToolBox()
  def apply(func: String): ClassInfo = this.synchronized {
    var clazz = clazzs.get(func)
    if (clazz == null) {
      val (className, classBody) = wrapClass(func)
      val zz = compile(prepareScala(className, classBody))
      val defaultMethod = zz.getDeclaredMethods.head
      val methods = zz.getDeclaredMethods
      clazz = ClassInfo(
        zz,
        zz.newInstance(),
        defaultMethod,
        methods = methods.map { m => (m.getName, m) }.toMap,
        func
      )
      clazzs.put(func, clazz)
      logInfo(s"dynamic load class => $clazz")
    }
    clazz
  }
  def compile(src: String): Class[_] = {
    val tree = toolBox.parse(src)
    toolBox.compile(tree).apply().asInstanceOf[Class[_]]
  }
  def prepareScala(className: String, classBody: String): String = {
    classBody + "\n" + s"scala.reflect.classTag[$className].runtimeClass"
  }
  def wrapClass(function: String): (String, String) = {
    val className = s"dynamic_class_${UUID.randomUUID().toString.replaceAll("-", "")}"
    val classBody =
      s"""
         |class $className{
         |  $function
         |}
            """.stripMargin
    (className, classBody)
  }
}
```

使用方法就灰常简单了我的大佬们。

```scala
val infos = ClassCreateUtils(
      """
        |def apply(name:String)=name.toUpperCase
      """.stripMargin
)
    
println(infos.defaultMethod.invoke(infos.instance,"dounine 本猪会一点点 spark"))
# 输出结果不用猜也知道是
DOUNINE 本猪会一点点 SPARK
# 也可以手动指定方法
println(infos.methods("apply").invoke(infos.instance,"dounine 本猪会一点点 spark"))
```

根据反射的方法信息生成`FunctionN`

```scala
object ScalaGenerateFuns {

  def apply(func: String): (AnyRef, Array[DataType], DataType) = {
    val (argumentTypes, returnType) = getFunctionReturnType(func)
    (generateFunction(func, argumentTypes.length), argumentTypes, returnType)
  }

  //获取方法的参数类型及返回类型
  private def getFunctionReturnType(func: String): (Array[DataType], DataType) = {
    val classInfo = ClassCreateUtils(func)
    val method = classInfo.defaultMethod
    val dataType = JavaTypeInference.inferDataType(method.getReturnType)._1
    (method.getParameterTypes.map(JavaTypeInference.inferDataType).map(_._1), dataType)
  }

  //生成22个Function
  def generateFunction(func: String, argumentsNum: Int): AnyRef = {
    lazy val instance = ClassCreateUtils(func).instance
    lazy val method = ClassCreateUtils(func).methods("apply")

    argumentsNum match {
      case 0 => new (() => Any) with Serializable with Logging {
        override def apply(): Any = {
          try {
            method.invoke(instance)
          } catch {
            case e: Exception =>
              logError(e.getMessage)
          }
        }
      }
      case 1 => new (Object => Any) with Serializable with Logging {
        override def apply(v1: Object): Any = {
          try {
            method.invoke(instance, v1)
          } catch {
            case e: Exception =>
              e.printStackTrace()
              logError(e.getMessage)
              null
          }
        }
      }
      case 2 => new ((Object, Object) => Any) with Serializable with Logging {
        override def apply(v1: Object, v2: Object): Any = {
          try {
            method.invoke(instance, v1, v2)
          } catch {
            case e: Exception =>
              logError(e.getMessage)
              null
          }
        }
      }
      //... 麻烦大佬自己去写剩下的20个了，这里装不下了，不然浏览器会崩溃的，然后电脑会重启的，为了大佬的电脑着想。
}
```

前戏我们都做完了，高潮的环节来了。

我们最后再照着`register`的实现方式，把我们动态`Function`注册给`Spark`

```scala
val ScalaReflection.Schema(dataType, nullable) = ScalaReflection.schemaFor[RT]
val inputTypes = Try(ScalaReflection.schemaFor[A1].dataType :: Nil).toOption
def builder(e: Seq[Expression]) = if (e.length == 1) {
  ScalaUDF(func, dataType, e, inputTypes.getOrElse(Nil), 
    Some(name), nullable, udfDeterministic = true)
}
functionRegistry.createOrReplaceTempFunction(name, builder)
```

1这句代码比较好理解，就是获取`RT`返回值类型，就是我们的returnType

2就是参数类型，对应的修改如下

```scala
val inputTypes = Try(argumentTypes.toList).toOption
```

3刚开始看到这个时候，我是一脸???，后来看源码才发现`builder`是一种自定类型,源码如下

```scala
type FunctionBuilder = Seq[Expression] => Expression
```

改造方式如下

```scala
def builder(e: Seq[Expression]) = ScalaUDF(rf, returnType, e, inputTypes.getOrElse(Nil), Some(name))
```

4看到这句的时候我以为简单了，直接使用`spark.sessionState.functionRegistry`发现编译不过，看到`private[sql]`这个作用域的时候有点崩溃，本来是想用下面的方式注册的。

```scala
val udf = UserDefinedFunction(rf, returnType, inputTypes).withName(name)
spark.udf.register(name, udf)
```

是小弟我想太多了，另辟捷径，做了那么多工作难道就白费了？

发现下面这句代码，瞬间找到了家的方向

```scala
functionRegistry.registerFunction(new FunctionIdentifier(name), builder)
```

到此，大猪的分析与编码已经完成，下面是今天给大哥的解决方案。

## 解决方案

方法实现可以通过查询sql得到，或者接口都渴以。

```scala
val spark = SparkSession
      .builder()
      .appName("test")
      .master("local[*]")
      .getOrCreate()

    val name = "hello"

    val (fun, argumentTypes, returnType) = ScalaSourceUDF(
      """
        |def apply(name:String)=name+" => hi"
        |""".stripMargin)

    val inputTypes = Try(argumentTypes.toList).toOption

    def builder(e: Seq[Expression]) = ScalaUDF(fun, returnType, e, inputTypes.getOrElse(Nil), Some(name))

    spark.sessionState.functionRegistry.registerFunction(new FunctionIdentifier(name), builder)

    val rdd = spark
      .sparkContext
      .parallelize(Array(("dounine", "20")))
      .map(x => Row.fromSeq(Array(x._1, x._2)))

    val types = StructType(
      Array(
        StructField("name", StringType),
        StructField("age", StringType)
      )
    )

    spark.createDataFrame(rdd, types).createTempView("log")

    spark.sql("select hello(name) from log").show(false)
```









