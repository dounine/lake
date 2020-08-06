---
description: >-
  我们现在有一个需求，将项目打包上传到`gitlab`或者`github`后，程序能自动部署，不用手动地去服务器中进行项目更新并运行，如何做到？这里我们可以使用`gitlab`与`github`的挂钩，挂钩的原理就是，每当我们有请求到`gitlab`与`github`服务器时，这时他俩会根据我们配置的挂钩地扯进行访问，`webhook`挂钩程序会一直监听着某个端口请求，一但收到他们发过来的请求，这时
---

# Webhook 部署自动化

## Gitlab

下载项目

```text
git clone https://github.com/dounine/gitlab-webhook.git
```

自行修改第9行读取密码文件的位置

```text
fs.readFile('/root/issp/gitlab-webhook/password.txt', 'utf8',....
```

修改第65行执行shell脚本位置

```text
cmd.get('/root/issp/docker/' + event.mode + '/run.sh',....
```

运行

```text
cd gitlab-webhook && ./start.sh
```

gitlab 配置

```text
URL：http://xxxxx:7777/webhook
Secret Token：password.txt里面的密码
```

## Github

下载项目

```text
git clone https://github.com/dounine/github-webhook.git
```

自行修改第3行密码文件的位置

```text
var secretPassword = 'abc123' //github secret安全密码
```

修改第7行执行shell脚本位置

```text
var bash = '/root/xxx/test.sh' //执行的脚本
```

运行

```text
cd github-webhook && ./start.sh
```

github配置

```text
Payload URL：http://xxxxx:7777/webhook
Secret：安全密码
```



