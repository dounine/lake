---
description: >-
  假设现在有A服务,B服务,外部使用RESTApi请求调用A服务，在请求头上有token字段，A服务使用完后，B服务也要使用，如何才能把token也转发到B服务呢？这里可以使用Feign的RequestInterceptor，但是直接使用一般情况下HttpServletRequest上下文对象是为空的，这里要怎么处理，请看下文。
---

# Fegin获取header异常解决方案

A服务FeginInterceptor

```java
@Configuration
public class FeginInterceptor implements RequestInterceptor {

    @Override
    public void apply(RequestTemplate requestTemplate) {
        Map<String,String> headers = getHeaders(getHttpServletRequest());
        for(String headerName : headers.keySet()){
            requestTemplate.header(headerName, getHeaders(getHttpServletRequest()).get(headerName));
        }
    }

    private HttpServletRequest getHttpServletRequest() {
        try {
            return ((ServletRequestAttributes) RequestContextHolder.getRequestAttributes()).getRequest();
        } catch (Exception e) {
            e.printStackTrace();
            return null;
        }
    }

    private Map<String, String> getHeaders(HttpServletRequest request) {
        Map<String, String> map = new LinkedHashMap<>();
        Enumeration<String> enumeration = request.getHeaderNames();
        while (enumeration.hasMoreElements()) {
            String key = enumeration.nextElement();
            String value = request.getHeader(key);
            map.put(key, value);
        }
        return map;
    }

}
```

A服务配置 bootstrap.yml

```yaml
hystrix:
  command:
    default:
      execution:
        isolation:
          strategy: SEMAPHORE
```

A服务build.gradle

```yaml
buildscript {
	ext{
		springBootVersion = '1.5.9.RELEASE'
	}

	repositories {
		mavenCentral()
	}
	dependencies {
		classpath("org.springframework.boot:spring-boot-gradle-plugin:${springBootVersion}")
		classpath "io.spring.gradle:dependency-management-plugin:0.5.6.RELEASE"
	}
}

apply plugin: 'java'
apply plugin: 'org.springframework.boot'
apply plugin: "io.spring.dependency-management"
version = '0.0.1-SNAPSHOT'
group 'com.dounine.test'

sourceCompatibility = 1.8

repositories {
	mavenLocal()
	mavenCentral()
}

ext {
	springCloudVersion = 'Dalston.SR2'
}

dependencies {
	compile('org.springframework.cloud:spring-cloud-starter-config')
	compile('org.springframework.cloud:spring-cloud-starter-eureka')
	compile('org.springframework.cloud:spring-cloud-starter-feign')
	compile group: 'org.aspectj', name: 'aspectjweaver', version: '1.8.13'
	compile('org.springframework.boot:spring-boot-starter-data-redis')
	testCompile('org.springframework.boot:spring-boot-starter-test')
}

dependencyManagement {
	imports {
		mavenBom "org.springframework.cloud:spring-cloud-dependencies:${springCloudVersion}"
	}
}

```

B服务Action

```java
@Autowired
    private HttpServletRequest servletRequest;

    public String test() {
        return servletRequest.getHeader("token");
    }
```

{% hint style="info" %}
如若B服务也要将header请求转发到其它服务，将A服务的配置也应用到B服务上即可
{% endhint %}

