---
description: >-
  我们可以使用scala
  shell做很多事情，比如测试一些demo，不用再打开idea那类那么重的编辑器，当然还有其它用法，像我们使用hbase有这样的问题，只是想测试hbase一些东西，但是每次连接hbase很慢，使用scala
  shell可以先把hbase连接池先创建好，需要测试什么样的代码直接放进去执行即可，即共享变量。
---

# Scala-Shell使用外部包方法

引用单个包

```text
scala
Welcome to Scala 2.12.7 (Java HotSpot(TM) 64-Bit Server VM, Java 1.8.0_181).
Type in expressions for evaluation. Or try :help.

scala> :require /path/something/commons.jar
```

使用脚本方式

```bash
#!/bin/bash
allJars=""
for file in /Users/lake/project/target/lib/*
do
  allJars="$allJars:$file"
done

scala -cp $allJars
```

执行脚本

```bash
scala
Welcome to Scala 2.12.7 (Java HotSpot(TM) 64-Bit Server VM, Java 1.8.0_181).
Type in expressions for evaluation. Or try :help.

scala>
```

