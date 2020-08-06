---
description: '可采用两种方式进行连接，`jmx`与`jstatd`,此文演示如何配置`jmx`进行连接调优'
---

# JMX 远程调优

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

### jvisualvm

![&#x6253;&#x5F00;&#x8F6F;&#x4EF6;](../../../.gitbook/assets/image%20%284%29.png)

![&#x6DFB;&#x52A0;&#x8FDC;&#x7A0B;&#x4E3B;&#x673A;&#x5730;&#x626F;](../../../.gitbook/assets/image%20%289%29.png)

![&#x6388;&#x6743;&#x914D;&#x7F6E;](../../../.gitbook/assets/image%20%281%29.png)

![&#x6210;&#x529F;&#x6548;&#x679C;](../../../.gitbook/assets/image%20%2815%29.png)

