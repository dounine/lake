---
description: >-
  Antlr4
  是一个强大的解析器的生成器，可以用来读取、处理、执行或翻译结构化文本，ANTLR可以从语法上来生成一个可以构建和遍历解析树的解析器，最出名的Spark计算引擎2.x就是用它来解析SQL的，是一个牛到没朋友的家伙。
---

# Spark 自定义解析SQL

## IDEA语法分析插件

下载 [antlr-v4-grammar-plugin](https://plugins.jetbrains.com/files/7358/53413/antlr-intellij-plugin-v4-1.9.zip?updateId=53413&pluginId=7358&family=intellij)

插件安装

![](../../../.gitbook/assets/image%20%2810%29.png)

g4语法文件使用的是sparkSQL的 [SqlBase.g4](https://github.com/apache/spark/blob/v2.3.1/sql/catalyst/src/main/antlr4/org/apache/spark/sql/catalyst/parser/SqlBase.g4) 文件进行改造的 [ArcSql.g4 ](https://raw.githubusercontent.com/dounine/arc/370dc33ff42cca73e2a01ee2156b5627ba086d95/src/main/resources/ArcSQL.g4)

右键选中 multiStatement 进行测试

![](../../../.gitbook/assets/image%20%281%29.png)

测试SQL语法树

![](../../../.gitbook/assets/image%20%2815%29.png)

生成解析配置

![](../../../.gitbook/assets/image%20%2822%29.png)



1. 右键`ArcSQL.g4`文件，在下拉选项`Configure ANTLR`即可出来。
2. 第一个`Output directory...`要写上输出代码的路径。
3. 比如把它放到当前项目的antlr4的包中`/dounine/github/arc/src/main/scala/com/dounine/arc/antlr4`
4. 右键`ArcSQL.g4`文件，选中`Generate ANTLR Recognizer`即可生成
5. 会生成这几个文件 ArcSQL.interp ArcSQL.tokens ArcSQLBaseListener ArcSQLBaseVisitor ArcSQLLexer ArcSQLLexer.interp ArcSQLLexer.tokens ArcSQLListener ArcSQLParser ArcSQLVisitor

## 测试

添加依赖

```text
compile group: 'org.antlr', name: 'antlr4', version: '4.7.2'
```

被动模式\(树解析到节点了通知\)

```scala
val loadLexer = new ArcSQLLexer(CharStreams.fromString(
      """
        select toUp(name) from log;
      """))
val tokens = new CommonTokenStream(loadLexer)
val parser = new ArcSQLParser(tokens)
val ctx = parser.multiStatement()
val listener = new ArcSQLBaseListener() {
      override def exitQuerySpecification(ctx: ArcSQLParser.QuerySpecificationContext): Unit = {
        val input = ctx.start.getTokenSource.asInstanceOf[ArcSQLLexer]._input
        val start = ctx.start.getStartIndex
        val stop = ctx.stop.getStopIndex
        val interval = new Interval(start, stop)
        val sqlText = input.getText(interval)
        println("表名 => " + ctx.tableAlias().strictIdentifier().getText)
        println("完整SQL =>" + sqlText)
      }
    }
ParseTreeWalker.DEFAULT.walk(listener, ctx)
```

输出\(在`ctx`中还有很多关于sql树信息\)

```scala
表名 => log
完整SQL =>select toUp(name) from log
```

主动模式\(主动去要数据\)

```scala
val vistor = new ArcSQLBaseVisitor[Unit] {

      override def visitQuerySpecification(ctx: QuerySpecificationContext): Unit = {
        val input = ctx.start.getTokenSource.asInstanceOf[ArcSQLLexer]._input
        val start = ctx.start.getStartIndex
        val stop = ctx.stop.getStopIndex
        val interval = new Interval(start, stop)
        val sqlText = input.getText(interval)
        println("表名 => " + ctx.tableAlias().strictIdentifier().getText)
        println("完整SQL =>" + sqlText)
      }
}
vistor.visit(ctx)
```







