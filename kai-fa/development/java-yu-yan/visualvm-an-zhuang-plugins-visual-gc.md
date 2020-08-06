---
description: >-
  在Java VisualVM这款java性能分析及调优工具如何加载插件？比如漂亮的`Visual
  GC`，**大猪**我比较喜欢这款漂亮的姑娘，当然了，如果觉得这是阻止了小伙伴们的进步，喜欢使用控制或者`jconsole`来分析的，那就可以退出此文了哈
---

# VisualVM安装plugins Visual GC

## 安装步骤

[Java VisualVM历史版本](https://visualvm.github.io/pluginscenters.html)

进行java的bin目录中

```text
cd $JAVA_HOME/bin
```

运行Java VisualVM

```text
./jvisualvm
```

菜单窗口Plugins

![](../../../.gitbook/assets/image%20%2818%29.png)

添加插件地扯

> 原地扯：[https://visualvm.java.net/uc/8u40/updates.xml.gz](https://visualvm.java.net/uc/8u40/updates.xml.gz)
>
> 新地扯：[https://visualvm.github.io/archive/uc/7u60/updates.xml.gz](https://visualvm.github.io/archive/uc/7u60/updates.xml.gz)

勾选使用即可

### 安装插件

这时`Available Plugins`就能显示出插件的列表了 我们勾选后进行安装即可

![](../../../.gitbook/assets/image%20%288%29.png)

双击我们程序的PID

![](../../../.gitbook/assets/image%20%284%29.png)



