# Pssh 一键管理

安装

```text
yum install pssh -y
```

编辑节点IP地扯

{% code title="/root/hosts.txt" %}
```text
1.host.com
2.host.com
3.host.com
```
{% endcode %}

节点执行命令

```text
pssh -H hosts.txt -P "ls"
```



