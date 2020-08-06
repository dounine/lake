---
description: Kibana 作为Elasticsearch优秀的可视化的开源分析工具，我们下面使用Docker结合进行最简单的上手演示。
---

# Docker Elasticsearch可视化Kibana

docker版本

```text
docker --version
Docker version 18.03.0-ce, build 0520e24
```

{% page-ref page="docker-elasticsearch-ke-shi-hua-kibana.md" %}

{% code title="docker-compose.yml" %}
```yaml
version: '2' 
services:
  elasticsearch:
    image: elasticsearch
    environment:
      - cluster.name=elasticsearch
    ports:
      - "9200:9200"
  kibana:
    image: kibana
    environment:
      SERVER_NAME: kibana
      ELASTICSEARCH_URL: http://192.168.1.186:9200
    ports:
      - "5601:5601"
```
{% endcode %}

启动elasticsearch与kibana

```yaml
docker-compose up
```

访问Kibana页面 [http://localhost:5601](http://localhost:5601)

![](../../../.gitbook/assets/image%20%283%29.png)

修改index pattern 为`*` 

选择`Time Filter field name`为第一个 

然后`Create` 点击`Discover`即可看到数据页面

![](../../../.gitbook/assets/image%20%2812%29.png)



