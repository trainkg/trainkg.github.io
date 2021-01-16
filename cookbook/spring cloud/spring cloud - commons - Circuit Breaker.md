## [Spring Cloud Circuit Breaker & Security](https://docs.spring.io/spring-cloud-commons/docs/current/reference/html/#spring-cloud-circuit-breaker)



[TOC]

### 1. [介绍](https://docs.spring.io/spring-cloud-commons/docs/current/reference/html/#introduction)

spring cloud circuit breaker提供了一个高度抽象. 提供了一致的API. (和JSR有点不一样，JSR现有规范，SPRING的处理方案是将现有的断路器实现进行统一)

#### 1.1 [实现列表](https://docs.spring.io/spring-cloud-commons/docs/current/reference/html/#supported-implementations)

支持列表

- [Resilience4J](https://github.com/resilience4j/resilience4j)
- [Sentinel](https://github.com/alibaba/Sentinel)
- [Spring Retry](https://github.com/spring-projects/spring-retry)

### 2. [核心概念](https://docs.spring.io/spring-cloud-commons/docs/current/reference/html/#core-concepts)

使用`CircuitBreakerFactory` 创建circuit breaker, 需要将实现放入`classpath` 中， 下面是使用范例

```java
@Service
public static class DemoControllerService {
    private RestTemplate rest;
    private CircuitBreakerFactory cbFactory;
	
    //自动注入
    public DemoControllerService(RestTemplate rest, CircuitBreakerFactory cbFactory) {
        this.rest = rest;
        this.cbFactory = cbFactory;
    }

    public String slow() {
        return cbFactory.create("slow").run(() -> rest.getForObject("/slow", String.class), throwable -> "fallback");
    }

}
```



`CircuitBreakerFactory.create` API 创建一个对象（`CircuitBreaker`） , `run` 方法持有一个`Supplier` 和 `function`, `Supplier` 就是run方法包裹的代码块，`function` 是代码块返回值， 如果出现异常会调用`fallack`, 如果不想回退，可以选择性处理（异常定制）

### 3. [Circuit Breakers In Reactive Code](https://docs.spring.io/spring-cloud-commons/docs/current/reference/html/#circuit-breakers-in-reactive-code)

如果application中有project reactor,  可以使用`ReactiveCircuitBreakerFactory`, 下面是例子

```java
@Service
public static class DemoControllerService {
    private ReactiveCircuitBreakerFactory cbFactory;
    private WebClient webClient;


    public DemoControllerService(WebClient webClient, ReactiveCircuitBreakerFactory cbFactory) {
        this.webClient = webClient;
        this.cbFactory = cbFactory;
    }

    public Mono<String> slow() {
        return webClient.get().uri("/slow").retrieve().bodyToMono(String.class).transform(
        it -> cbFactory.create("slow").run(it, throwable -> return Mono.just("fallback")));
    }
}
```

`ReactiveCircuitBreakerFactory.create` 创建一个对象`ReactiveCircuitBreaker`, run 参数是 Mono或者是Flux

### 4. [配置](https://docs.spring.io/spring-cloud-commons/docs/current/reference/html/#configuration)

可以通过创建`Customizer` 的Bean配置断路器，这个接口只有一个`customize` ,持有一个对象进行客户定制化。如何配置一个断路器需要看对应的实现

- [Resilience4J](https://docs.spring.io/spring-cloud-commons/spring-cloud-circuitbreaker/current/reference/html/spring-cloud-circuitbreaker.html#configuring-resilience4j-circuit-breakers)
- [Sentinal](https://github.com/alibaba/spring-cloud-alibaba/blob/master/spring-cloud-alibaba-docs/src/main/asciidoc/circuitbreaker-sentinel.adoc#circuit-breaker-spring-cloud-circuit-breaker-with-sentinel—configuring-sentinel-circuit-breakers)
- [Spring Retry](https://docs.spring.io/spring-cloud-commons/spring-cloud-circuitbreaker/current/reference/html/spring-cloud-circuitbreaker.html#configuring-spring-retry-circuit-breakers)



### 5. [CachedRandomPropertySource](https://docs.spring.io/spring-cloud-commons/docs/current/reference/html/#cachedrandompropertysource)

在配置参数中加入一个随机配置参数。如果期望系统重启之后，随机数还是之前的数值， 需要利用缓存

`cachedrandom.[yourkey].[type]` 

> 这个是否应该看缓存方案？

```properties
myrandom=${cachedrandom.appname.value}
```



### 6. 权限(TODO)

#### 6.1 单点登陆

单点登陆支持功能在spring boot中实现支持

##### 6.1.1 客户端令牌中继

如果使用的是OAth2 Client模式， 那么他在spring boot会有一个`OAuth2ClientContext` 在request scope[spring bean 级别]， 可以创建`OAuth2RestTemplate`

 并且注入一个`OAuth2ProtectedResourceDetails` , 他在访问下游服务的时候，会传递这个令牌。

##### 6.1.2 服务器令牌中继

如果应用有`@EnableResourceServer` , 如果期望传递令牌到后续服务， 可以使用`RestTemplate` 连接下游服务，需要注意的是如何正确创建`resttemplate`.

如果服务使用`UserInfoTokenServices` 

### 7 附件

Spring cloud common 相关联的全量配置参考 [the Appendix page](https://docs.spring.io/spring-cloud-commons/docs/current/reference/html/appendix.html).

