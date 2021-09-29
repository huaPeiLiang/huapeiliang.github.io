---
title: Stream 项目 - Gateway 及灰度发布
description: 本篇文章将介绍：需求分析、Nacos 请求隔离、Gateway 配置示例、匹配规则、自定义拦截器、灰度发布
categories:
- 架构之路
---

> 学海无涯，心存高远。

## 需求分析

- 开发阶段大家服务全部注册到开发环境的 nacos，需要避免请求被别人截获。
- 正式上线后，每次的迭代版本不能直接让所有用户切换过来。

## Nacos 请求隔离

通过 feign 进行服务间调用时，可以借助 nacos 的服务名进行请求隔离。在每个服务的配置文件中都加上版本号。

```
VERSION=
spring.application.name=xxx-${VERSION}
```

以 IntelliJ Idea 为例，在"Environment variables"环境变量里添加 VERSION 值：

```
VERSION=hpl
```

这样每次服务间调用只会转发到自己的服务上。

## Gateway 配置示例

```
spring:
  cloud:
    gateway:
      routes:
      - id: after_route
        uri: https://example.org
        predicates:
        - Cookie=mycookie,mycookievalue
```

## 匹配规则

**The After Route Predicate Factory**

将指定日期后的请求进行匹配
```
spring:
  cloud:
    gateway:
      routes:
      - id: after_route
        uri: https://example.org
        predicates:
        - After=2017-01-20T17:42:47.789-07:00[America/Denver]
```

**The Cookie Route Predicate Factory**

根据指定 cookie 参数进行匹配
```
spring:
  cloud:
    gateway:
      routes:
      - id: cookie_route
        uri: https://example.org
        predicates:
        - Cookie=chocolate, ch.p
```

**The Header Route Predicate Factory**

根据指定 header 参数进行匹配
```
spring:
  cloud:
    gateway:
      routes:
      - id: header_route
        uri: https://example.org
        predicates:
        - Header=X-Request-Id, \d+
```

**The Host Route Predicate Factory**

根据 Host 规则进行匹配
```
spring:
  cloud:
    gateway:
      routes:
      - id: host_route
        uri: https://example.org
        predicates:
        - Host=**.somehost.org,**.anotherhost.org
```

**The Method Route Predicate Factory**

根据 Method 规则进行匹配
```
spring:
  cloud:
    gateway:
      routes:
      - id: method_route
        uri: https://example.org
        predicates:
        - Method=GET,POST
```

**The Path Route Predicate Factory**

根据路径进行匹配
```
spring:
  cloud:
    gateway:
      routes:
      - id: path_route
        uri: https://example.org
        predicates:
        - Path=/red/{segment},/blue/{segment}
```

**The Weight Route Predicate Factory**

根据权重进行匹配
```
spring:
  cloud:
    gateway:
      routes:
      - id: weight_high
        uri: https://weighthigh.org
        predicates:
        - Weight=group1, 8
      - id: weight_low
        uri: https://weightlow.org
        predicates:
        - Weight=group1, 2
```

## 自定义拦截器

自定义一个拦截器，通过 getOrder 方法设置它的执行顺序，filter 方法里写具体的处理逻辑。
```
@Component
public class GrayFilter implements GlobalFilter, Ordered {
     @Override
    public int getOrder() {
        return 1;
    }

    @Override
    public Mono<Void> filter(ServerWebExchange exchange, GatewayFilterChain chain) {
        ...
    }
}
```

要想自定义的拦截器生效定义一个配置类，代码如下：

```
@Configuration
public class GrayFilterConfiguration {

    @Bean
    @ConditionalOnMissingBean({GrayFilter.class})
    public GrayFilter gatewayLoadBalancerGrayFilter() {
        return new GrayFilter();
    }

}
```

## 灰度发布

定义好路由规则和拦截器，然后就可以在拦截器中实现根据版本号进行路由转发。

在项目的配置文件中配置 nacos 元数据信息：
```
spring.cloud.nacos.discovery.metadata.Version=${VERSION}
```

gateway 中获取项目的元数据信息，根据元数据中的 Version 进行请求转发：
```
private Response<ServiceInstance> getServiceInstanceResponseByVersion(List<ServiceInstance> instances, HttpHeaders headers) {
        String versionNo = headers.getFirst("Version");
        Map<String,String> versionMap = new HashMap<>();
        versionMap.put("Version",versionNo);
        final Set<Map.Entry<String,String>> attributes =
                Collections.unmodifiableSet(versionMap.entrySet());
        ServiceInstance serviceInstance = null;
        for (ServiceInstance instance : instances) {
            Map<String,String> metadata = instance.getMetadata();
            if(metadata.entrySet().containsAll(attributes)){
                serviceInstance = instance;
                break;
            }
        }
        if(serviceInstance == null){
            return new EmptyResponse();
        }
        return new DefaultResponse(serviceInstance);
    }
```

前端只需要在请求的 header 中添加"Version"参数，就可以转发到对应的服务上。










