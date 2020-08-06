---
description: >-
  有没有遇到这么一些问题,开发在本地测试没问题，将项目打包到线上计算出来的时间不是少了8个小时就是多了8个小时，这是因为容器的系统默认时间跟我们中国的时间对不上，所以才会有这样的问题。
---

# Docker 解决容器时区时间不一致

最傻瓜也最方便的处理方式

```text
docker run -v /etc/timezone:/etc/timezone -v /etc/localtime:/etc/localtime -ti centos bash
```

以上将宿主机的时间与本地时间绑定到容器中，这样时间就会跟宿主机一样了。

```text
/etc/timezone 时区
/etc/localtime 时间
```

验证时间是否正确,在控制台输入以下命令即可

```text
date
```

