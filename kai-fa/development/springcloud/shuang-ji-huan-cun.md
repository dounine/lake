---
description: >-
  在springcloud中我们可以使用spring-boot-starter-data-redis已经为我们处理好分布式缓存，但是我们还是不满足于只存在于网络中传输的缓存，我们现在来扩展成本地加Redis双级缓存，这样就可以减少网络传输带来的传输效率。
---

# 双级缓存

## 使用

以下是针对已经整理好的项目进行直接使用

打包安装项目 [springcloud-twocache](https://github.com/dounine/springcloud-twocache.git)

```text
git clone https://github.com/dounine/spring-cloud.git
cd spring-cloud
gradle install -xtest
```

在项目中引用

```text
dependencies {
    compile('com.dounine.twocache:springcloud-twocache:0.0.1-SNAPSHOT')
}
```

application.yml 添加配置

```yaml
spring:
  redis:
    host: localhost
    port: 6379
twocache:
  enable: true
  redis:
    topic: 项目名
```

java代码中使用\(与spring cache使用缓存一样\)

```java
@Cacheable(cacheNames = "user",key = "#userId")
public String queryUser(@PathVariable String userId) {
...
}
```

## 源码讲解

IpV4.java 节点IP获取工具

```java
import java.net.Inet4Address;
import java.net.UnknownHostException;

public final class IpV4 {
    private static String node;
    static {
        try {
            node = Inet4Address.getLocalHost().getHostAddress();
        } catch (UnknownHostException e) {
            e.printStackTrace();
        }
    }
    public static final String get(){
        return node;
    }
}
```

NotifyMsg.java Redis消息通知包装对象

```java
public class NotifyMsg implements Serializable {
    private NotifyType notifyType;
    private String cacheName;
    private String node;
    private Object key;
    private Object result;

    public NotifyMsg(NotifyType notifyType,String node,Object key,Object result){
        this.node = node;
        this.notifyType = notifyType;
        this.key = key;
        this.result = result;
    }
    
    // get set ...
}
```

NotifyType.java Redis缓存通知类型

```java
public enum NotifyType {
    PUT,
    EVICT,
    CLEAR
}
```

RedisAndLocalCache.java Redis本地缓存重写

```java
public class RedisAndLocalCache implements Cache {

    private ConcurrentHashMap<Object,ValueWrapper> local = new ConcurrentHashMap<>();
    private RedisCache redisCache;
    private TwoLevelCacheManager cacheManager;
    private String node;

    public RedisAndLocalCache(TwoLevelCacheManager twoLevelCacheManager,RedisCache redisCache,String node){
        this.cacheManager = twoLevelCacheManager;
        this.redisCache = redisCache;
        this.node = node;
    }

    @Override
    public String getName() {
        return redisCache.getName();
    }

    @Override
    public Object getNativeCache() {
        return redisCache.getNativeCache();
    }

    @Override
    public ValueWrapper get(Object key) {
        ValueWrapper valueWrapper = local.get(key);
        if(valueWrapper!=null){
            return valueWrapper;
        }else{
            valueWrapper = redisCache.get(key);
            if(valueWrapper!=null){
                local.put(key,valueWrapper);
            }
            return valueWrapper;
        }
    }

    @Override
    public <T> T get(Object key, Class<T> type) {
        ValueWrapper valueWrapper = local.get(key);
        if(valueWrapper!=null){
            return (T)valueWrapper.get();
        }else{
            valueWrapper = redisCache.get(key);
            if(valueWrapper!=null){
                local.put(key,valueWrapper);
            }
            return (T)valueWrapper.get();
        }
    }

    @Override
    public <T> T get(Object key, Callable<T> valueLoader) {
        return null;
    }

    @Override
    public void put(Object key, Object value) {
        this.local.put(key,new SimpleValueWrapper(value));
        this.redisCache.put(key,value);
        this.notifyNodes(new NotifyMsg(NotifyType.PUT,node,key,value));
    }

    private void notifyNodes(NotifyMsg notifyType){
        notifyType.setCacheName(redisCache.getName());
        cacheManager.publishMessage(notifyType);
    }


    @Override
    public ValueWrapper putIfAbsent(Object key, Object value) {
        return null;
    }

    @Override
    public void evict(Object key) {
        redisCache.evict(key);
        this.notifyNodes(new NotifyMsg(NotifyType.EVICT,node,key,null));
    }

    public void clearLocal(){
        local.clear();
    }

    @Override
    public void clear() {
        redisCache.clear();
        this.notifyNodes(new NotifyMsg(NotifyType.CLEAR,node,null,null));
    }
}
```

TwoCacheConfig.java Starter 自动配置类

```java
@Configuration
@ConditionalOnMissingBean(CacheManager.class)
@ConditionalOnBean({RedisTemplate.class})
@ConditionalOnProperty(name = "twocache.enable",havingValue = "true")
@EnableCaching
public class TwoCacheConfig {

    @Value("${twocache.redis.topic:towcache}")
    private String topic;
    @Value("${server.port}")
    private Integer port;

    @Bean
    @ConditionalOnMissingBean(JedisConnectionFactory.class)
    JedisConnectionFactory jedisConnectionFactory() {
        return new JedisConnectionFactory();
    }

    @Bean
    RedisMessageListenerContainer container(RedisConnectionFactory connectionFactory, MessageListenerAdapter listenerAdapter){
        RedisMessageListenerContainer container = new RedisMessageListenerContainer();
        container.setConnectionFactory(connectionFactory);
        container.addMessageListener(listenerAdapter,new PatternTopic(topic));
        return container;
    }

    @Bean
    MessageListenerAdapter listenerAdapter(final TwoLevelCacheManager cacheManager){
        return new MessageListenerAdapter(new MessageListener() {
            @Override
            public void onMessage(Message message, byte[] pattern) {
                try {
                    String topic = new String(message.getChannel(),"utf-8");
                    cacheManager.receiver(message.getBody());
                } catch (UnsupportedEncodingException e) {
                    e.printStackTrace();
                }
            }
        });
    }

    @Bean
    public TwoLevelCacheManager cacheManager(RedisTemplate redisTemplate){
        return new TwoLevelCacheManager(redisTemplate,topic,port);
    }
}
```

TwoLevelCacheManager.java 双级缓存管理器

```java
public class TwoLevelCacheManager extends RedisCacheManager {

    private String topic;
    private RedisTemplate<String,Object> redisTemplate;
    private Integer port;
    private String node = IpV4.get();

    public TwoLevelCacheManager(RedisTemplate<String,Object> redisTemplate,String topic, Integer port){
        super(redisTemplate);
        this.redisTemplate = redisTemplate;
        this.topic = topic;
        this.port = port;
    }

    @Override
    protected Cache decorateCache(Cache cache) {
        return new RedisAndLocalCache(this,(RedisCache) cache,node+":"+port);
    }

    protected void publishMessage(NotifyMsg notifyMsg){
        this.redisTemplate.convertAndSend(topic,notifyMsg);
    }

    public void receiver(byte[] body){
        NotifyMsg notifyMsg = (NotifyMsg)this.redisTemplate.getDefaultSerializer().deserialize(body);
        RedisAndLocalCache cache = (RedisAndLocalCache) this.getCache(notifyMsg.getCacheName());
        if(cache!=null){
            if(!notifyMsg.getNode().equals(node+":"+port)){
                if(notifyMsg.getNotifyType().equals(NotifyType.CLEAR)){
                    cache.clearLocal();
                }else if(notifyMsg.getNotifyType().equals(NotifyType.PUT)){
                    cache.put(notifyMsg.getKey(),notifyMsg.getResult());
                }else if(notifyMsg.getNotifyType().equals(NotifyType.EVICT)){
                    cache.evict(notifyMsg.getKey());
                }
            }else{
//                LOGGER.error("消息从自身发送,忽略处理");
            }
        }
    }
}
```

