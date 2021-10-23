## [spring cloud gateway](https://docs.spring.io/spring-cloud-gateway/docs/current/reference/html/)

### 版本要求

Spring 5, Spring Boot 2

### 安装

添加Maven依赖.

```xml
<dependency>
<groupId>org.springframework.cloud</groupId>
<artifactId>spring-cloud-starter-gateway</artifactId>
</dependency>	
```

spring 开关配置

```properties
spring.cloud.gateway.enabled=false|true	
```

> spring gateway 不是基于传统的httpservlet技术，是基于NIO的多路复用，所以不可以作为war运行。



### 术语说明

- **Route**: 构建网关基础信息. 包含 ID,目标地址,  命中条件， 过滤器集合. 
- **Predicate**:  JAVA8 Predicate 特性. 输入参数是 [Spring Framework `ServerWebExchange`](https://docs.spring.io/spring/docs/5.0.x/javadoc-api/org/springframework/web/server/ServerWebExchange.html).  用于匹配http中的任何信息
- **Filter**: [`GatewayFilter`](https://github.com/spring-cloud/spring-cloud-gateway/tree/2.2.x/spring-cloud-gateway-server/src/main/java/org/springframework/cloud/gateway/filter/GatewayFilter.java) 实例， 在发送给下游请求的时候，可以修改request或者response中的内容。

### 工作方式

提供一个high level 概述图

![Spring Cloud Gateway Diagram](https://docs.spring.io/spring-cloud-gateway/docs/2.2.9.RELEASE/reference/html/images/spring_cloud_gateway_diagram.png)



总体来说gateway的设计方式并没有变化，重点是他采用了NIO技术，提高负载能力，网关是压力的节点。

### 配置Route Predicate 和 Filter

两种方式配置Predicate 和 filter

* shortcuts 便捷模式  （常用，简化写法）
* fully expanded arguments  完全模式， 所有信息都一一对应标准的写好

#### [Shortcut Configuration](https://docs.spring.io/spring-cloud-gateway/docs/2.2.9.RELEASE/reference/html/#shortcut-configuration)

该配置方式通过filter name进行辨识

```yaml
spring:
  cloud:
    gateway:
      routes:
      - id: after_route
        uri: https://example.org
        predicates:
        - Cookie=mycookie,mycookievalue
```

上面的例子匹配方式使用 `Cookie` Route ，并且带有两个参数

#### [Fully Expanded Arguments](https://docs.spring.io/spring-cloud-gateway/docs/2.2.9.RELEASE/reference/html/#fully-expanded-arguments)

```yaml
spring:
  cloud:
    gateway:
      routes:
      - id: after_route
        uri: https://example.org
        predicates:
        - name: Cookie
          args:
            name: mycookie
            regexp: mycookievalue	
```

这个效果和上面的shortcut配置是一样的

### Route Predicate Factories

首先 spring cloud gateway 有多个构建Predicate工厂，是基于Spring WebFlux实现 handlerMapping实现的。Predicate工厂是可以通过 and逻辑合并的。

#### The After Route Predicate Factory

After router predicate factory 有一个参数， `datetime`  Java 类型`zoneDateTime` , 可以匹配特定的时间

```yaml
spring:
  cloud:
    gateway:
      routes:
      - id: after_route
        uri: https://example.org
        predicates:
        - After=2017-01-20T17:42:47.789-07:00[America/Denver]
```

这个匹配 `2017-01-20T17:42:47.789-07:00` 这个时间点之后的任何请求

#### [The Before Route Predicate Factory](https://docs.spring.io/spring-cloud-gateway/docs/2.2.9.RELEASE/reference/html/#the-before-route-predicate-factory)

```yaml
spring:
  cloud:
    gateway:
      routes:
      - id: before_route
        uri: https://example.org
        predicates:
        - Before=2017-01-20T17:42:47.789-07:00[America/Denver]
```

类似After, 匹配指定时间点之前的所有请求

#### [The Between Route Predicate Factory](https://docs.spring.io/spring-cloud-gateway/docs/2.2.9.RELEASE/reference/html/#the-between-route-predicate-factory)

```yaml
spring:
  cloud:
    gateway:
      routes:
      - id: between_route
        uri: https://example.org
        predicates:
        - Between=2017-01-20T17:42:47.789-07:00[America/Denver], 2017-01-21T17:42:47.789-07:00[America/Denver]
```

时间段内请求匹配

#### [The Cookie Route Predicate Factory](https://docs.spring.io/spring-cloud-gateway/docs/2.2.9.RELEASE/reference/html/#the-cookie-route-predicate-factory)

```yaml
spring:
  cloud:
    gateway:
      routes:
      - id: cookie_route
        uri: https://example.org
        predicates:
        - Cookie=chocolate, ch.p
```

匹配请求信息中的Cookie信息，两个参数，`name` 和 `regexp` Java 正则表达式

#### [The Header Route Predicate Factory](https://docs.spring.io/spring-cloud-gateway/docs/2.2.9.RELEASE/reference/html/#the-header-route-predicate-factory)

两个参数，`name` 和 `regexp` Java 正则表达式 匹配请求头信息

```yaml
spring:
  cloud:
    gateway:
      routes:
      - id: header_route
        uri: https://example.org
        predicates:
        - Header=X-Request-Id, \d+
```

#### [The Host Route Predicate Factory](https://docs.spring.io/spring-cloud-gateway/docs/2.2.9.RELEASE/reference/html/#the-host-route-predicate-factory)

只有一个参数 `patterns`,  这个参数是ant-style风格的， 使用`.` 作为分隔符， 匹配请求方host.

```yaml
spring:
  cloud:
    gateway:
      routes:
      - id: host_route
        uri: https://example.org
        predicates:
        - Host=**.somehost.org,**.anotherhost.org
```

ant 形式的模糊匹配，比如 {sub}.somehost.org

#### [The Method Route Predicate Factory](https://docs.spring.io/spring-cloud-gateway/docs/2.2.9.RELEASE/reference/html/#the-method-route-predicate-factory)

`http method`, 只有一个参数`methods`, 可以同时匹配多个支持的方法， 匹配http请求中的请求方法

```yaml
spring:
  cloud:
    gateway:
      routes:
      - id: method_route
        uri: https://example.org
        predicates:
        - Method=GET,POST
```

#### [The Path Route Predicate Factory](https://docs.spring.io/spring-cloud-gateway/docs/2.2.9.RELEASE/reference/html/#the-path-route-predicate-factory)

请求路径映射，两个参数 `PathMatcher` `patterns`, 还有一个可选项， `matchOptionalTrailingSeparator`

```yaml
spring:
  cloud:
    gateway:
      routes:
      - id: path_route
        uri: https://example.org
        predicates:
        - Path=/red/{segment},/blue/{segment}
```

其实所有的Route匹配中，所有的匹配项后续都可以类似如下的方式获取

```java
Map<String, String> uriVariables = ServerWebExchangeUtils.getPathPredicateVariables(exchange);

String segment = uriVariables.get("segment");	
```

#### [The Query Route Predicate Factory](https://docs.spring.io/spring-cloud-gateway/docs/2.2.9.RELEASE/reference/html/#the-query-route-predicate-factory)

请求参数匹配  两个参数, `param` 和 `regexp` 

```yaml
spring:
  cloud:
    gateway:
      routes:
      - id: remoteaddr_route
        uri: https://example.org
        predicates:
        - RemoteAddr=192.168.1.1/24
```

#### [The RemoteAddr Route Predicate Factory](https://docs.spring.io/spring-cloud-gateway/docs/2.2.9.RELEASE/reference/html/#the-remoteaddr-route-predicate-factory)

这个匹配一个数组，最少有一个元素， 匹配请求IP地址。

```yaml
spring:
  cloud:
    gateway:
      routes:
      - id: remoteaddr_route
        uri: https://example.org
        predicates:
        - RemoteAddr=192.168.1.1/24
```

其中 192.168.1.1 是IP, 24 是子网掩码

#### [ The Weight Route Predicate Factory](https://docs.spring.io/spring-cloud-gateway/docs/2.2.9.RELEASE/reference/html/#the-weight-route-predicate-factory)

权重匹配 两个参数 group 和 weight  (int 类型)， 需要结合其他的Route

```yaml
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

80% 请求转向 `weighthigh`, 20%转向`weightlow`

#### [Modifying the Way Remote Addresses Are Resolved](https://docs.spring.io/spring-cloud-gateway/docs/2.2.9.RELEASE/reference/html/#modifying-the-way-remote-addresses-are-resolved)

当spring cloud gateway在代理层后方的时候 （由网络代理请求网关）， 会导致网关获取不到真实的IP. 

可以自定义的方式是，通过设定一个`RemoteAddressResolver`, 那么网关将会使用默认的地址

具体参考官方文档，有点复杂

### Filter 工厂

Router filters 是运行修改请求和响应的

#### [The `AddRequestHeader` `GatewayFilter` Factory](https://docs.spring.io/spring-cloud-gateway/docs/2.2.9.RELEASE/reference/html/#the-addrequestheader-gatewayfilter-factory)

```yaml
spring:
  cloud:
    gateway:
      routes:
      - id: add_request_header_route
        uri: https://example.org
        predicates:
        - Path=/red/{segment}
        filters:
        - AddRequestHeader=X-Request-Red, Blue-{segment}
```

两个参数， name and value,  为下游的添加请求头信息。

#### [The `AddRequestParameter` `GatewayFilter` Factory](https://docs.spring.io/spring-cloud-gateway/docs/2.2.9.RELEASE/reference/html/#the-addrequestparameter-gatewayfilter-factory)

```yaml
spring:
  cloud:
    gateway:
      routes:
      - id: add_response_header_route
        uri: https://example.org
        predicates:
        - Host: {segment}.myhost.org
        filters:
        - AddResponseHeader=foo, bar-{segment}
```

两个参数， name and value,  为下游的添加请求参数信息。

#### [The `DedupeResponseHeader` `GatewayFilter` Factory](https://docs.spring.io/spring-cloud-gateway/docs/2.2.9.RELEASE/reference/html/#the-deduperesponseheader-gatewayfilter-factory)

两个参数 ， name 和 可选参数 strategy, name 可以使用空格间隔， 删除请求头中的重复数据

```yaml
spring:
  cloud:
    gateway:
      routes:
      - id: dedupe_response_header_route
        uri: https://example.org
        filters:
        - DedupeResponseHeader=Access-Control-Allow-Credentials Access-Control-Allow-Origin
```

#### [ The Hystrix `GatewayFilter` Factory](https://docs.spring.io/spring-cloud-gateway/docs/2.2.9.RELEASE/reference/html/#hystrix)

服务熔断器，建议使用spring cloud circuitBreaker

首先需要启用`spring-cloud-starter-netflix-hystrix`,  一个参数 name ,是hystrix命令名称

```yaml
spring:
  cloud:
    gateway:
      routes:
      - id: hystrix_route
        uri: https://example.org
        filters:
        - Hystrix=myCommandName
```

#### [Spring Cloud CircuitBreaker GatewayFilter Factory](https://docs.spring.io/spring-cloud-gateway/docs/2.2.9.RELEASE/reference/html/#spring-cloud-circuitbreaker-filter-factory)

spring cloud CircuitBreaker  提供两个类库的封装  Hystrix and Resilience4J， 建议使用Resilience4J.

启用filter需要引入对应的JAR

`spring-cloud-starter-circuitbreaker-reactor-resilience4j`  or `spring-cloud-starter-netflix-hystrix`

```yaml
spring:
  cloud:
    gateway:
      routes:
      - id: circuitbreaker_route
        uri: https://example.org
        filters:
        - CircuitBreaker=myCircuitBreaker
```

#### [Tripping The Circuit Breaker On Status Codes](https://docs.spring.io/spring-cloud-gateway/docs/2.2.9.RELEASE/reference/html/#circuit-breaker-status-codes)

```yaml
spring:
  cloud:
    gateway:
      routes:
      - id: circuitbreaker_route
        uri: lb://backing-service:8088
        predicates:
        - Path=/consumingServiceEndpoint
        filters:
        - name: CircuitBreaker
          args:
            name: myCircuitBreaker
            fallbackUri: forward:/inCaseOfFailureUseThis
            statusCodes:
              - 500
              - "NOT_FOUND"
```

如果filter没有通过，将会设定对应的httpstatus为500， fallback指降级处理，如果不能通过则转到下一个请求。

#### [The `FallbackHeaders` `GatewayFilter` Factory](https://docs.spring.io/spring-cloud-gateway/docs/2.2.9.RELEASE/reference/html/#fallback-headers)

```yaml
spring:
  cloud:
    gateway:
      routes:
      - id: ingredients
        uri: lb://ingredients
        predicates:
        - Path=//ingredients/**
        filters:
        - name: CircuitBreaker
          args:
            name: fetchIngredients
            fallbackUri: forward:/fallback
      - id: ingredients-fallback
        uri: http://localhost:9994
        predicates:
        - Path=/fallback
        filters:
        - name: FallbackHeaders
          args:
            executionExceptionTypeHeaderName: Test-Header
```

如上例子中当执行出现异常的时候，会转向 `/fallback` , 然后在请头信息添加对应的失败异常信息和root cause. 详细参考官方文档

#### [The `MapRequestHeader` `GatewayFilter` Factory](https://docs.spring.io/spring-cloud-gateway/docs/2.2.9.RELEASE/reference/html/#the-maprequestheader-gatewayfilter-factory)

两个参数 `fromHeader` to `header` 

```yaml
spring:
  cloud:
    gateway:
      routes:
      - id: map_request_header_route
        uri: https://example.org
        filters:
        - MapRequestHeader=Blue, X-Request-Red
```

对于请求头包含Blue 的，之后的请求添加X-Request-Red

#### [The `PrefixPath` `GatewayFilter` Factory](https://docs.spring.io/spring-cloud-gateway/docs/2.2.9.RELEASE/reference/html/#the-prefixpath-gatewayfilter-factory)

```yaml
spring:
  cloud:
    gateway:
      routes:
      - id: prefixpath_route
        uri: https://example.org
        filters:
        - PrefixPath=/mypath
```

只有一个参数 `PrefixPath`, 用来修改请求路径的 , 比如这里 `/hello` 会被修改为 `/mypath/hello`

#### [The `PreserveHostHeader` `GatewayFilter` Factory](https://docs.spring.io/spring-cloud-gateway/docs/2.2.9.RELEASE/reference/html/#the-preservehostheader-gatewayfilter-factory)

没有参数

```yaml
spring:
  cloud:
    gateway:
      routes:
      - id: preserve_host_route
        uri: https://example.org
        filters:
        - PreserveHostHeader
```

这个filter比较特殊，标记当前请求保留主机Host信息。

#### [The `RequestRateLimiter` `GatewayFilter` Factory](https://docs.spring.io/spring-cloud-gateway/docs/2.2.9.RELEASE/reference/html/#the-requestratelimiter-gatewayfilter-factory)

这个Filter使用一个`RateLimiter` 实现类，用于觉得当前的请求是否允许去处理， 如果不行则需设定http status 为 429 - Too many requests 然后返回。

可选参数是`keyResolver`， 类型是`keyResolver`, 依赖可以使用 spring EL表达式设定 `{@myKeyResolver}`， 当前默认的对象是`PrincipalNameKeyResolver`. 或者自我实现

```java
public interface KeyResolver {
    Mono<String> resolve(ServerWebExchange exchange);
}
```

配置信息参考官方文档

##### [The Redis `RateLimiter`](https://docs.spring.io/spring-cloud-gateway/docs/2.2.9.RELEASE/reference/html/#the-redis-ratelimiter)

首先需要引入`spring-boot-starter-data-redis-reactive`, 因为需要使用 Redis， 

The `redis-rate-limiter.replenishRate` 控制每秒请求数，不会删除请求. 比如请求100个，每秒开放60个，后续放到后续的时间段，这样就造成了延迟

The `redis-rate-limiter.burstCapacity` 期望控制每秒最大的请求数，如果是0则block所有的请求

The `redis-rate-limiter.requestedTokens` 限制每个请求花费的时间，单位秒，默认是1秒

```yaml
spring:
  cloud:
    gateway:
      routes:
      - id: requestratelimiter_route
        uri: https://example.org
        filters:
        - name: RequestRateLimiter
          args:
            redis-rate-limiter.replenishRate: 10  -- 每秒期望10
            redis-rate-limiter.burstCapacity: 20  -- 每秒最大20，两次的超出会导致出现http 429 异常
            redis-rate-limiter.requestedTokens: 1 -- 需要在1s内完成请求
```

自定义的一个`keyResolver`, 如下 

> 生产环境不推荐从request里面获取User信息

```java
@Bean
KeyResolver userKeyResolver() {
    return exchange -> Mono.just(exchange.getRequest().getQueryParams().getFirst("user"));
}
```

同时也可以自己实现`RateLimiter`接口, 并且加入到spring 配置中, 配置中使用方式如下

```yaml
spring:
  cloud:
    gateway:
      routes:
      - id: requestratelimiter_route
        uri: https://example.org
        filters:
        - name: RequestRateLimiter
          args:
            rate-limiter: "#{@myRateLimiter}"
            key-resolver: "#{@userKeyResolver}"
```

#### [ The `RedirectTo` `GatewayFilter` Factory](https://docs.spring.io/spring-cloud-gateway/docs/2.2.9.RELEASE/reference/html/#the-redirectto-gatewayfilter-factory)

有两参数 `status` 和 `url`, 状态使用300系列状态吗， url需要是一个有效的地址， 本地和相对地址需要区别

```yaml
spring:
  cloud:
    gateway:
      routes:
      - id: prefixpath_route
        uri: https://example.org
        filters:
        - RedirectTo=302, https://acme.org
```

#### [The `RemoveRequestHeader` GatewayFilter Factory](https://docs.spring.io/spring-cloud-gateway/docs/2.2.9.RELEASE/reference/html/#the-removerequestheader-gatewayfilter-factory)

删除请求头中信息，参数name

```yaml
spring:
  cloud:
    gateway:
      routes:
      - id: removerequestheader_route
        uri: https://example.org
        filters:
        - RemoveRequestHeader=X-Request-Foo  --后续请求将无法访问这个Header信息
```

#### [`RemoveResponseHeader` `GatewayFilter` Factory](https://docs.spring.io/spring-cloud-gateway/docs/2.2.9.RELEASE/reference/html/#removeresponseheader-gatewayfilter-factory)

这个是删除响应头信息中的内容， 参数name 

```yaml
spring:
  cloud:
    gateway:
      routes:
      - id: removerequestparameter_route
        uri: https://example.org
        filters:
        - RemoveRequestParameter=red
```

#### [The `RewritePath` `GatewayFilter` Factory](https://docs.spring.io/spring-cloud-gateway/docs/2.2.9.RELEASE/reference/html/#the-rewritepath-gatewayfilter-factory)

这个是灵活定义重写的地址，前面有一个可以添加前缀的，这个更加灵活，参数 规则 `regexp`+替换 `replacement`

```yaml
spring:
  cloud:
    gateway:
      routes:
      - id: rewritepath_route
        uri: https://example.org
        predicates:
        - Path=/red/**
        filters:
        - RewritePath=/red(?<segment>/?.*), $\{segment} 
        #segment为匹配到的对象
```

#### [`RewriteLocationResponseHeader` `GatewayFilter` Factory](https://docs.spring.io/spring-cloud-gateway/docs/2.2.9.RELEASE/reference/html/#rewritelocationresponseheader-gatewayfilter-factory)

```yaml
spring:
  cloud:
    gateway:
      routes:
      - id: rewritelocationresponseheader_route
        uri: http://example.org
        filters:
        - RewriteLocationResponseHeader=AS_IN_REQUEST, Location, ,
```

It takes `stripVersionMode`, `locationHeaderName`, `hostValue`, and `protocolsRegex` parameters.

For example, for a request of `POST api.example.com/some/object/name`, the `Location` response header value of `object-service.prod.example.net/v2/some/object/id` is rewritten as `api.example.com/some/object/id`.

按照上面的配置，URL 将被重写。

`stripVersionMode 三个可能设定: `NEVER_STRIP`, `AS_IN_REQUEST` (default), and `ALWAYS_STRIP`.

- `NEVER_STRIP`: The version is not stripped, even if the original request path contains no version.
- `AS_IN_REQUEST` The version is stripped only if the original request path contains no version.
- `ALWAYS_STRIP` The version is always stripped, even if the original request path contains version.

The `hostValue` parameter ， 可选

如果设定话，则使用location的`host:port` 替换， 如果没有提供则使用请求方的信息， 所以上面的例子被替换为请求的`api.example.com`

The `protocolsRegex` parameter 必须是一个有效的正则表达式， 字符串类型， 默认值 `http|https|ftp|ftps` 意思就是四个任意一个

如果不匹配，啥也不干.

#### [The `RewriteResponseHeader` `GatewayFilter` Factory](https://docs.spring.io/spring-cloud-gateway/docs/2.2.9.RELEASE/reference/html/#the-rewriteresponseheader-gatewayfilter-factory)

```yaml
spring:
  cloud:
    gateway:
      routes:
      - id: rewriteresponseheader_route
        uri: https://example.org
        filters:
        - RewriteResponseHeader=X-Response-Red, , password=[^&]+, password=***
        # 配置name???
```

 takes `name`, `regexp`, and `replacement` parameters.

For a header value of `/42?user=ford&password=omg!what&flag=true`, it is set to `/42?user=ford&password=***&flag=true` after making the downstream request. You must use `$\` to mean `$` because of the YAML specification.

密码替换 ***

#### [The `SaveSession` `GatewayFilter` Factory](https://docs.spring.io/spring-cloud-gateway/docs/2.2.9.RELEASE/reference/html/#the-savesession-gatewayfilter-factory)

```yaml
spring:
  cloud:
    gateway:
      routes:
      - id: save_session
        uri: https://example.org
        predicates:
        - Path=/foo/**
        filters:
        - SaveSession
        # ???
```

强制执行`websession::save` 

#### [The `SecureHeaders` `GatewayFilter` Factory](https://docs.spring.io/spring-cloud-gateway/docs/2.2.9.RELEASE/reference/html/#the-secureheaders-gatewayfilter-factory)

The `SecureHeaders` `GatewayFilter` factory adds a number of headers to the response, per the recommendation made in [this blog post](https://blog.appcanary.com/2017/http-security-headers.html).

安全相关，对响应头做出一些限制， 后续结合Http协议查看相关设定

The following headers (shown with their default values) are added:

- `X-Xss-Protection:1 (mode=block`)
- `Strict-Transport-Security (max-age=631138519`)
- `X-Frame-Options (DENY)`
- `X-Content-Type-Options (nosniff)`
- `Referrer-Policy (no-referrer)`
- `Content-Security-Policy (default-src 'self' https:; font-src 'self' https: data:; img-src 'self' https: data:; object-src 'none'; script-src https:; style-src 'self' https: 'unsafe-inline)'`
- `X-Download-Options (noopen)`
- `X-Permitted-Cross-Domain-Policies (none)`

**默认值修改方式**

o change the default values, set the appropriate property in the `spring.cloud.gateway.filter.secure-headers` namespace. The following properties are available:

- `xss-protection-header`
- `strict-transport-security`
- `x-frame-options`
- `x-content-type-options`
- `referrer-policy`
- `content-security-policy`
- `x-download-options`
- `x-permitted-cross-domain-policies`

**关闭相关头设定**

To disable the default values set the `spring.cloud.gateway.filter.secure-headers.disable` property with comma-separated values. The following example shows how to do so:

```
spring.cloud.gateway.filter.secure-headers.disable=x-frame-options,strict-transport-security
```

#### [The `SetPath` `GatewayFilter` Factory](https://docs.spring.io/spring-cloud-gateway/docs/2.2.9.RELEASE/reference/html/#the-setpath-gatewayfilter-factory)

```yaml
spring:
  cloud:
    gateway:
      routes:
      - id: setpath_route
        uri: https://example.org
        predicates:
        - Path=/red/{segment}
        filters:
        - SetPath=/{segment}
```

也是一种修改请求路径的方式

#### [The `SetRequestHeader` `GatewayFilter` Factory](https://docs.spring.io/spring-cloud-gateway/docs/2.2.9.RELEASE/reference/html/#the-setrequestheader-gatewayfilter-factory)

```yaml
spring:
  cloud:
    gateway:
      routes:
      - id: setrequestheader_route
        uri: https://example.org
        filters:
        - SetRequestHeader=X-Request-Red, Blue
```

takes `name` and `value` parameters. 

这个主要功能是替换头信息内容，不是添加， 还支持变量处理

```yaml
spring:
  cloud:
    gateway:
      routes:
      - id: setrequestheader_route
        uri: https://example.org
        predicates:
        - Host: {segment}.myhost.org
        filters:
        - SetRequestHeader=foo, bar-{segment}
```

URL中的变量是可以使用的..

#### [The `SetResponseHeader` `GatewayFilter` Factory](https://docs.spring.io/spring-cloud-gateway/docs/2.2.9.RELEASE/reference/html/#the-setresponseheader-gatewayfilter-factory)

设置响应头信息，takes `name` and `value` parameters 

```yaml
spring:
  cloud:
    gateway:
      routes:
      - id: setresponseheader_route
        uri: https://example.org
        filters:
        - SetResponseHeader=X-Response-Red, Blue
```

当服务响应设定为 `X-Response-Red:1234` 会被替换为 `X-Response-Red:Blue`,  同时也支持变量设定

```yaml
spring:
  cloud:
    gateway:
      routes:
      - id: setresponseheader_route
        uri: https://example.org
        predicates:
        - Host: {segment}.myhost.org
        filters:
        - SetResponseHeader=foo, bar-{segment}
```

#### [The `SetStatus` `GatewayFilter` Factory](https://docs.spring.io/spring-cloud-gateway/docs/2.2.9.RELEASE/reference/html/#the-setstatus-gatewayfilter-factory)

takes a single parameter, `status`	, 必须是一个有效的`httpstatus`

```yaml
spring:
  cloud:
    gateway:
      routes:
      - id: setstatusstring_route
        uri: https://example.org
        filters:
        - SetStatus=BAD_REQUEST
      - id: setstatusint_route
        uri: https://example.org
        filters:
        - SetStatus=401
```

上面的例子， response将会被设定为401

```yaml
spring:
  cloud:
    gateway:
      set-status:
        original-status-header-name: original-http-status
```

上面的例子，可以将代理情况下的`http`状态返回给请求方

#### [The `StripPrefix` `GatewayFilter` Factory](https://docs.spring.io/spring-cloud-gateway/docs/2.2.9.RELEASE/reference/html/#the-stripprefix-gatewayfilter-factory)

takes one parameter, `parts`, 这个参数代表数量， 参考下面的例子

```yaml
spring:
  cloud:
    gateway:
      routes:
      - id: nameRoot
        uri: https://nameservice
        predicates:
        - Path=/name/**
        filters:
        - StripPrefix=2
```

When a request is made through the gateway to `/name/blue/red`, the request made to `nameservice` looks like `nameservice/red`.

因为设定的是2，所以前两个/name/blue会被忽略，直接访问 https://nameservice/red

#### [The Retry `GatewayFilter` Factory](https://docs.spring.io/spring-cloud-gateway/docs/2.2.9.RELEASE/reference/html/#the-retry-gatewayfilter-factory)

重试特性

The `Retry` `GatewayFilter` factory supports the **following parameters:**

- `retries`: 失败尝试次数 int.

- `statuses`: 尝试失败之后的状态码

- `methods`: 允许重试的`http`请求类型，通常不应该为delete, post 提供重试

- `series`: The series of status codes to be retried, represented by using `org.springframework.http.HttpStatus.Series`.

- `exceptions`: A list of thrown exceptions that should be retried.

- `backoff`: The configured exponential backoff for the retries. Retries are performed after a backoff interval of `firstBackoff * (factor ^ n)`, where `n` is the iteration. If `maxBackoff` is configured, the maximum backoff applied is limited to `maxBackoff`. If `basedOnPreviousValue` is true, the backoff is calculated byusing `prevBackoff * factor`.

  backoff 是避免接口进行多次重试，导致服务长时间不能恢复， 具体影响公式参考上面的说明



**The following defaults** are configured for `Retry` filter, if enabled:

- `retries`: 3次

- `series`: 5XX series

- `methods`: GET method

- `exceptions`: `IOException` and `TimeoutException`

- `backoff`: disabled

  

```yaml
spring:
  cloud:
    gateway:
      routes:
      - id: retry_test
        uri: http://localhost:8080/flakey
        predicates:
        - Host=*.retry.com
        filters:
        - name: Retry
          args:
            retries: 3
            statuses: BAD_GATEWAY
            methods: GET,POST
            backoff:
              firstBackoff: 10ms
              maxBackoff: 50ms
              factor: 2
              basedOnPreviousValue: false
```

#### [The `RequestSize` `GatewayFilter` Factory](https://docs.spring.io/spring-cloud-gateway/docs/2.2.9.RELEASE/reference/html/#the-requestsize-gatewayfilter-factory)

The filter takes a `maxSize` parameter, 和数据大小定义一致，KB, MB etc. 默认是 B （BYTE）

```YAML
spring:
  cloud:
    gateway:
      routes:
      - id: request_size_route
        uri: http://localhost:8080/upload
        predicates:
        - Path=/upload
        filters:
        - name: RequestSize
          args:
            maxSize: 5000000
```

对于上传链接大小的一个限制，将限制逻辑和上传业务剥离开来，更加独立， 如果超出了大小限制，则响应状态码为 `413 Payload Too Large` 

#### [The `SetRequestHostHeader` `GatewayFilter` Factory](https://docs.spring.io/spring-cloud-gateway/docs/2.2.9.RELEASE/reference/html/#the-setrequesthostheader-gatewayfilter-factory)

```yaml
spring:
  cloud:
    gateway:
      routes:
      - id: set_request_host_header_route
        uri: http://localhost:8080/headers
        predicates:
        - Path=/headers
        filters:
        - name: SetRequestHostHeader
          args:
            host: example.org
```

 The filter takes a `host` parameter.

#### [Modify a Request Body `GatewayFilter` Factory](https://docs.spring.io/spring-cloud-gateway/docs/2.2.9.RELEASE/reference/html/#modify-a-request-body-gatewayfilter-factory)

这种方式的filter只能通过JAVA配置

```java
@Bean
public RouteLocator routes(RouteLocatorBuilder builder) {
    return builder.routes()
        .route("rewrite_request_obj", r -> r.host("*.rewriterequestobj.org")
            .filters(f -> f.prefixPath("/httpbin")
                .modifyRequestBody(String.class, Hello.class, MediaType.APPLICATION_JSON_VALUE,
                    (exchange, s) -> return Mono.just(new Hello(s.toUpperCase())))).uri(uri))
        .build();
}

static class Hello {
    String message;

    public Hello() { }

    public Hello(String message) {
        this.message = message;
    }

    public String getMessage() {
        return message;
    }

    public void setMessage(String message) {
        this.message = message;
    }
}
```

#### [Modify a Response Body `GatewayFilter` Factory](https://docs.spring.io/spring-cloud-gateway/docs/2.2.9.RELEASE/reference/html/#modify-a-response-body-gatewayfilter-factory)

```java
@Bean
public RouteLocator routes(RouteLocatorBuilder builder) {
    return builder.routes()
        .route("rewrite_response_upper", r -> r.host("*.rewriteresponseupper.org")
            .filters(f -> f.prefixPath("/httpbin")
                .modifyResponseBody(String.class, String.class,
                    (exchange, s) -> Mono.just(s.toUpperCase()))).uri(uri))
        .build();
}
```

#### [ Default Filters](https://docs.spring.io/spring-cloud-gateway/docs/2.2.9.RELEASE/reference/html/#default-filters)

如果需要针对所有路由设定全局的过滤器，可以使用`spring.cloud.gateway.default-filters` 指定一个filter列表

```yaml
spring:
  cloud:
    gateway:
      default-filters:
      - AddResponseHeader=X-Response-Default-Red, Default-Blue
      - PrefixPath=/httpbin
```

### [Global Filters](https://docs.spring.io/spring-cloud-gateway/docs/2.2.9.RELEASE/reference/html/#global-filters)

#### [Combined Global Filter and `GatewayFilter` Ordering](https://docs.spring.io/spring-cloud-gateway/docs/2.2.9.RELEASE/reference/html/#gateway-combined-global-filter-and-gatewayfilter-ordering)

全局过滤器是一个链状，如果需要排序可以实现spring `ordered` 接口，

Example 59. ExampleConfiguration.java

```java
@Bean
public GlobalFilter customFilter() {
    return new CustomGlobalFilter();
}

public class CustomGlobalFilter implements GlobalFilter, Ordered {

    @Override
    public Mono<Void> filter(ServerWebExchange exchange, GatewayFilterChain chain) {
        log.info("custom global filter");
        return chain.filter(exchange);
    }

    @Override
    public int getOrder() {
        return -1;
    }
}
```

#### [Forward Routing Filter](https://docs.spring.io/spring-cloud-gateway/docs/2.2.9.RELEASE/reference/html/#forward-routing-filter)

支持`forward` 协议， such as `forward:///localendpoint`

`ServerWebExchangeUtils.GATEWAY_REQUEST_URL_ATTR` 加工后的URL地址

`ServerWebExchangeUtils.GATEWAY_ORIGINAL_REQUEST_URL_ATTR` 加工前的URL地址

#### [The `LoadBalancerClient` Filter](https://docs.spring.io/spring-cloud-gateway/docs/2.2.9.RELEASE/reference/html/#the-loadbalancerclient-filter)

针对请求lb开头的，例如`lb://myservice`,  这个将会导致后面使用 `LoadBalancerClient` 去查找`myservice`. 并且替换响应的host和port. 

`ServerWebExchangeUtils.GATEWAY_ORIGINAL_REQUEST_URL_ATTR` 没有修改之前的URL, `ServerWebExchangeUtils.GATEWAY_SCHEME_PREFIX_ATTR` 

```yaml
spring:
  cloud:
    gateway:
      routes:
      - id: myRoute
        uri: lb://service
        predicates:
        - Path=/service/**
```

上面的例子为一个服务配置了负载均衡策略

> 如果service在LB 客户端找不到的话， 会返回503错误， 也可以配置 `spring.cloud.gateway.loadbalancer.use404=true` 替换404

#### [The `ReactiveLoadBalancerClientFilter`](https://docs.spring.io/spring-cloud-gateway/docs/2.2.9.RELEASE/reference/html/#reactive-loadbalancer-client-filter)

**推荐使用**

读取上下文`ServerWebExchangeUtils.GATEWAY_REQUEST_URL_ATTR`参数，如果URL里面包含`lb` ,那么将使用spring cloud `ReactorLoadBalancer` 

```yaml
spring:
  cloud:
    gateway:
      routes:
      - id: myRoute
        uri: lb://service
        predicates:
        - Path=/service/**
```

 By default, when a service instance cannot be found by the `ReactorLoadBalancer`, a `503` is returned. You can configure the gateway to return a `404` by setting `spring.cloud.gateway.loadbalancer.use404=true`.

#### [The Netty Routing Filter](https://docs.spring.io/spring-cloud-gateway/docs/2.2.9.RELEASE/reference/html/#the-netty-routing-filter)

查找 `ServerWebExchangeUtils.GATEWAY_REQUEST_URL_ATTR`  ， 如果存在 `http` or `https` ，  It uses the Netty `HttpClient` to make the downstream proxy request. The response is put in the `ServerWebExchangeUtils.CLIENT_RESPONSE_ATTR` exchange attribute for use in a later filter. (There is also an experimental `WebClientHttpRoutingFilter` that performs the same function but does not require Netty.)

#### [The Netty Write Response Filter](https://docs.spring.io/spring-cloud-gateway/docs/2.2.9.RELEASE/reference/html/#the-netty-write-response-filter)

The `NettyWriteResponseFilter` runs if there is a Netty `HttpClientResponse` in the `ServerWebExchangeUtils.CLIENT_RESPONSE_ATTR` exchange attribute. It runs after all other filters have completed and writes the proxy response back to the gateway client response. (There is also an experimental `WebClientWriteResponseFilter` that performs the same function but does not require Netty.)

查看response，在其他filter全部执行完毕之后， 重新写入一个代理响应给网关客户端

### [The `RouteToRequestUrl` Filter](https://docs.spring.io/spring-cloud-gateway/docs/2.2.9.RELEASE/reference/html/#the-routetorequesturl-filter)

查看 `ServerWebExchangeUtils.GATEWAY_ROUTE_ATTR`属性, the `RouteToRequestUrlFilter` 启用.  构建一个新的URL, 基于之前的请求，但是URL变量更新为  `ServerWebExchangeUtils.GATEWAY_ROUTE_ATTR` 属性内容.  新的URL被保存在  `ServerWebExchangeUtils.GATEWAY_REQUEST_URL_ATTR` exchange attribute.

If the URI has a scheme prefix, such as `lb:ws://serviceid`, the `lb` scheme is stripped from the URI and placed in the `ServerWebExchangeUtils.GATEWAY_SCHEME_PREFIX_ATTR` for use later in the filter chain.

#### [The Websocket Routing Filter](https://docs.spring.io/spring-cloud-gateway/docs/2.2.9.RELEASE/reference/html/#the-websocket-routing-filter)

启用条件： If the URL located in the `ServerWebExchangeUtils.GATEWAY_REQUEST_URL_ATTR` exchange attribute has a `ws` or `wss` scheme

可以启用负载均衡 such as `lb:ws://serviceid`.

```yaml
spring:
  cloud:
    gateway:
      routes:
      # SockJS route
      - id: websocket_sockjs_route
        uri: http://localhost:3001
        predicates:
        - Path=/websocket/info/**
      # Normal Websocket route
      - id: websocket_route
        uri: ws://localhost:3001
        predicates:
        - Path=/websocket/**
```

#### [The Gateway Metrics Filter](https://docs.spring.io/spring-cloud-gateway/docs/2.2.9.RELEASE/reference/html/#the-gateway-metrics-filter)

To enable gateway metrics, add spring-boot-starter-actuator as a project dependency. Then, by default, the gateway metrics filter runs as long as the property `spring.cloud.gateway.metrics.enabled` is not set to `false`. This filter adds a timer metric named `gateway.requests` with the following tags:

- `routeId`: The route ID.
- `routeUri`: The URI to which the API is routed.
- `outcome`: The outcome, as classified by [HttpStatus.Series](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/http/HttpStatus.Series.html).
- `status`: The HTTP status of the request returned to the client.
- `httpStatusCode`: The HTTP Status of the request returned to the client.
- `httpMethod`: The HTTP method used for the request.

#### [Marking An Exchange As Routed](https://docs.spring.io/spring-cloud-gateway/docs/2.2.9.RELEASE/reference/html/#marking-an-exchange-as-routed)

After the gateway has routed a `ServerWebExchange`, it marks that exchange as “routed” by adding `gatewayAlreadyRouted` to the exchange attributes. Once a request has been marked as routed, other routing filters will not route the request again, essentially skipping the filter. There are convenience methods that you can use to mark an exchange as routed or check if an exchange has already been routed.

- `ServerWebExchangeUtils.isAlreadyRouted` takes a `ServerWebExchange` object and checks if it has been “routed”.
- `ServerWebExchangeUtils.setAlreadyRouted` takes a `ServerWebExchange` object and marks it as “routed”.