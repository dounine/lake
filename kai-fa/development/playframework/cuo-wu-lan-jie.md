---
description: 在一般的web框架中、比如springboot、或者springcloud。再或者struct2都会有统一错误拦截器。playframework也不例外
---

# 错误拦截

application.conf 添加配置

```text
play.http.errorHandler = "ErrorHandler"
```

自定义 ErrorHandler.scala 类

```scala
import java.util.concurrent.{CompletableFuture, CompletionStage}

import exception.AuthException
import play.http.HttpErrorHandler
import play.libs.Json
import play.mvc
import play.mvc.{Http, Results}

class ErrorHandler extends HttpErrorHandler {

  override def onClientError(request: Http.RequestHeader, statusCode: Int, message: String): CompletionStage[mvc.Result] = {
    CompletableFuture.completedFuture(
      Results.status(statusCode, "A client error occurred: " + message))
  }

  override def onServerError(request: Http.RequestHeader, exception: Throwable): CompletionStage[mvc.Result] = {
    CompletableFuture.completedFuture(
      exception match {
        case e: AuthException =>
          val json = Json.newObject()
          json.put("code", "error")
          json.put("msg", e.getMessage)
          Results.badRequest(json)
        case default@_ =>
          val json = Json.newObject()
          json.put("code", "error")
          json.put("msg", default.getMessage)
          Results.internalServerError(json)
      }
    )
  }

}
```

定义授权错误类

```scala
class AuthException(msg: String) extends Exception(msg) {}
```

UserController.scala 测试

```scala
def login: Action[AnyContent] = Action {
    request =>

      if(true){
        throw new AuthException("登录失效")
      }
```

{% hint style="info" %}
`Json.newObject()` 返回的数据类型为`application/json`
{% endhint %}

