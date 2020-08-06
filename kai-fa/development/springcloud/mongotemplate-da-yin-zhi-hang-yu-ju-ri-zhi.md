---
description: '在调试MongoTemplate的时候，我们无法确定我们写的方法是否能发出正确的语句,这时候我们需要将语句给打印出来。'
---

# MongoTemplate打印执行语句日志

在application.yml文件中增加如下配置即可

```text
logging:
  level:
    org.springframework.data.mongodb.core.MongoTemplate: DEBUG
```

