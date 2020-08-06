---
description: >-
  如果你使用Bazel的默认配置、它是将缓存放到/tmp目录下的、不到几分钟、你再次刷新项目的时候就没了、这时你就得重新下载构建了、这里教大家两种配置保存cache的方式。
---

# Bazel - 编译加速

## 保存本地磁盘

在用户目录创建一个文件 `~/.bazelrc` 内容如下

```
$ build --disk_cache=/path/cache
```

{% hint style="info" %}
 缓存内容不会自动删除、磁盘会越来越大!!!
{% endhint %}

## 保存网络

需要依赖`docker`环境

```text
$ docker run -d -v /path/cache:/data -p 9090:8080 -p 9092:9092 --name bazel-remote-cache buchgr/bazel-remote-cache
```

在用户目录创建一个文件 `~/.bazelrc` 内容如下

```text
build --remote_cache=http://127.0.0.1:9090
build --remote_upload_local_results=true
```

{% hint style="info" %}
依赖 docker
{% endhint %}



