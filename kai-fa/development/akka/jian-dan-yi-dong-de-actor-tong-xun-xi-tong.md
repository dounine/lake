# 简单易懂的Actor通讯系统

添加依赖

```text
compile group: 'com.typesafe.akka', name: 'akka-actor_2.12', version: '2.5.21'
compile group: 'com.typesafe.akka', name: 'akka-remote_2.12', version: '2.5.21'
```

定义消息协议

```scala
object Messages {

  case class Hello(content: String)
  case class World(content: String)

}
```

定义常量

```scala
object Cons {

  val ResourceManagerName = "ResourceManagerName"
  val NodeManagerName = "NodeManagerName"
  val ResourceActor = "ResourceActor"
  val NodeActor = "NodeActor"

}
```

服务

```scala
import akka.actor._
import com.typesafe.config.{Config, ConfigFactory}

class MyResourceManager() extends Actor {
  override def receive: Receive = {
    case Messages.Hello(content: String) => {

      sender() ! Messages.World("服务器回调")
    }
  }
}

object MyResourceManagerMain {
  def main(args: Array[String]): Unit = {
    val str: String =
      """
        |akka.actor.provider = "akka.remote.RemoteActorRefProvider"
        |akka.actor.warn-about-java-serializer-usage = off
        |akka.remote.netty.tcp.hostname = localhost
        |akka.remote.netty.tcp.port = 20000
      """.stripMargin
    val conf: Config = ConfigFactory.parseString(str)
    val actorSystem = ActorSystem(Cons.ResourceManagerName, conf)
    actorSystem.actorOf(Props(new MyResourceManager()), Cons.ResourceActor)
  }
}
```

节点

```scala
import akka.actor.{Actor, ActorSelection, ActorSystem, Props}
import com.typesafe.config.{Config, ConfigFactory}

class MyNodeManager(resourceHost: String = "localhost", resourcePort: Int = 20000) extends Actor {

  var resourceManager: ActorSelection = _

  override def preStart(): Unit = {
    resourceManager = context.actorSelection(s"akka.tcp://${Cons.ResourceManagerName}@$resourceHost:$resourcePort/user/${Cons.ResourceActor}")

    resourceManager ! Messages.Hello("haha")
  }

  override def receive: Receive = {
    case Messages.World(content: String) => {
      println(content)
    }
  }
}

object MyNodeManagerMain {
  def main(args: Array[String]): Unit = {
    val str: String =
      """
        |akka.actor.provider = "akka.remote.RemoteActorRefProvider"
        |akka.actor.warn-about-java-serializer-usage = off
        |akka.remote.netty.tcp.hostname = localhost
        |akka.remote.netty.tcp.port = 20001
      """.stripMargin
    val conf: Config = ConfigFactory.parseString(str)
    val actorSystem = ActorSystem(Cons.NodeManagerName, conf)
    actorSystem.actorOf(Props(new MyNodeManager()), Cons.NodeActor)
  }
}
```

启动 MyResourceManager

启动 MyNodeManager

```scala
[INFO] [02/21/2019 01:13:02.734] [main] [akka.remote.Remoting] Starting remoting
[INFO] [02/21/2019 01:13:02.878] [main] [akka.remote.Remoting] Remoting started; listening on addresses :[akka.tcp://NodeManagerName@localhost:20001]
[INFO] [02/21/2019 01:13:02.879] [main] [akka.remote.Remoting] Remoting now listens on addresses: [akka.tcp://NodeManagerName@localhost:20001]
服务器回调
```



