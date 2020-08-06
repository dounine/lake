---
description: '可采用两种方式进行连接，`jmx`与`jstatd`,此文演示如何配置`jmx`进行连接调优'
---

# JMX远程调优

## 非认证连接

```text
$ java \
 -Dcom.sun.management.jmxremote.authenticate=false \
 -Dcom.sun.management.jmxremote.port=5555 \
 -Dcom.sun.management.jmxremote.ssl=false \
 -Djava.rmi.server.hostname=test.domain.dounine.com \
 -jar test-jar-1.0.jar
```

## 启用用户认证

jmxremote.access 用户权限

```text
dounine readwrite
```

jmxremote.password 密码文件

```text
dounine password
```

增加程序访问权限

```bash
$ chmod 400 ./jmxremote.access
$ chmod 400 ./jmxremote.password
```

运行程序

```text
$ java \
 -Dcom.sun.management.jmxremote.authenticate=true \
 -Dcom.sun.management.jmxremote.port=5555 \
 -Dcom.sun.management.jmxremote.access.file=/path/jmxremote.access \
 -Dcom.sun.management.jmxremote.password.file=/path/jmxremote.password \
 -Dcom.sun.management.jmxremote.ssl=false \
 -Djava.rmi.server.hostname=test.domain.dounine.com \
 -jar test-jar-1.0.jar
```

启动`jvisualvm`或者`jconsole`

```text
jvisualvm
# 或者
jconsole
```



![](http://upload-images.jianshu.io/upload_images/9028759-1fe88e88e8a8291c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)





