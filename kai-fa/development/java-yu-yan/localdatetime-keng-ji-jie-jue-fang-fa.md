---
description: 在用新的`Date API`的时候大家有木有遇到很多坑，这里就告诉大家如何解决字符串转换`LocalDateTime`中的方法
---

# LocalDateTime 坑及解决方法

## 常规

```java
String valueIn = "2018-01-24 10:13:52";
DateTimeFormatter DATETIME = DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm:ss");
LocalDateTime ldt = LocalDateTime.parse(valueIn, DATETIME);
System.out.println(ldt);
```

输出结果

```java
2018-01-24T10:13:52
```

真实想要的结果

```java
2018-01-24T10:13:52:00
```

可以这么修改

```java
DateTimeFormatter DATETIME = DateTimeFormatter.ofPattern("yyyy-MM-dd'T'HH:mm:ss");
```

如果后面带毫秒数呢？我们可以这么写

```java
DateTimeFormatter DATETIME = DateTimeFormatter.ofPattern("yyyy-MM-dd'T'HH:mm:ss.SSS");
```

## SpringBoot

以下配置处于SpringBoot上下文扫描范围中

```java
@ControllerAdvice
public class VControllerAdvice extends ValidateControllerAdvice{
  private static final DateTimeFormatter LOCAL_DATE_TIME = DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm:ss");
  private static final DateTimeFormatter LOCAL_DATE = DateTimeFormatter.ofPattern("yyyy-MM-dd");

    @InitBinder
    public void initBinder(WebDataBinder binder) {
        binder.registerCustomEditor(LocalDate.class, new PropertyEditorSupport() {
            @Override
            public void setAsText(String text) throws IllegalArgumentException {
                this.setValue(LocalDate.parse(text, LOCAL_DATE));
            }
        });
        binder.registerCustomEditor(LocalDateTime.class, new PropertyEditorSupport() {
            @Override
            public void setAsText(String text) throws IllegalArgumentException {
                this.setValue(LocalDateTime.parse(text, LOCAL_DATE_TIME));
            }
        });
    }
}
```



