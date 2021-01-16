## [Spring cloud LoadBalancer](https://docs.spring.io/spring-cloud-commons/docs/current/reference/html/#spring-cloud-loadbalancer)

[TOC]

spring cloud 提供了自己的客户端和负载均衡抽象以及实现，针对负载均衡机制， `ReactiveLoadBalancer`  支持 **Round-Robin-based** 和 **Random** ， 可以通过接口`ServiceInstanceListSupplier`  获取服务节点列表



### 1. [Switching between the load-balancing algorithms](https://docs.spring.io/spring-cloud-commons/docs/current/reference/html/#switching-between-the-load-balancing-algorithms)

`ReactiveLoadBalancer` 默认实现是 `RoundRobinLoadBalancer` , 如果要换实现，可以参考[custom LoadBalancer configurations mechanism](https://docs.spring.io/spring-cloud-commons/docs/current/reference/html/#custom-loadbalancer-configuration).

举个例子： 使用`RandomLoadBalancer` 实现

```java
public class CustomLoadBalancerConfiguration {

    @Bean
    ReactorLoadBalancer<ServiceInstance> randomLoadBalancer(Environment environment,
            LoadBalancerClientFactory loadBalancerClientFactory) {
        String name = environment.getProperty(LoadBalancerClientFactory.PROPERTY_NAME);
        return new RandomLoadBalancer(loadBalancerClientFactory
                .getLazyProvider(name, ServiceInstanceListSupplier.class),
                name);
    }
}
```



### 2. [Spring Cloud LoadBalancer integrations](https://docs.spring.io/spring-cloud-commons/docs/current/reference/html/#spring-cloud-loadbalancer-integrations)

为了让负载均衡更加简单，提供了 `ReactorLoadBalancerExchangeFilterFunction` ,可以在`WebClient` ,`BlockingLoadBalancerClient`,`RestTemplate` 中使用他

具体用例可以参考下面

- [Spring RestTemplate as a Load Balancer Client](https://docs.spring.io/spring-cloud-commons/docs/current/reference/html/#rest-template-loadbalancer-client)
- [Spring WebClient as a Load Balancer Client](https://docs.spring.io/spring-cloud-commons/docs/current/reference/html/#webclinet-loadbalancer-client)
- [Spring WebFlux WebClient with `ReactorLoadBalancerExchangeFilterFunction`](https://docs.spring.io/spring-cloud-commons/docs/current/reference/html/#webflux-with-reactive-loadbalancer)



### 3. [Spring Cloud LoadBalancer Caching](https://docs.spring.io/spring-cloud-commons/docs/current/reference/html/#loadbalancer-caching)

#### 3.1[ ](https://docs.spring.io/spring-cloud-commons/docs/current/reference/html/#caffeine-backed-loadbalancer-cache-implementation)[Caffeine](https://github.com/ben-manes/caffeine)-backed LoadBalancer Cache Implementation

`com.github.ben-manes.caffeine:caffeine` 

#### 3.2 [Default LoadBalancer Cache Implementation](https://docs.spring.io/spring-cloud-commons/docs/current/reference/html/#default-loadbalancer-cache-implementation)

默认的cache实现， `DefaultLoadBalancerCache` , 自动注册

#### 3.3 [LoadBalancer Cache Configuration](https://docs.spring.io/spring-cloud-commons/docs/current/reference/html/#loadbalancer-cache-configuration)

1. `spring.cloud.loadbalancer.cache.ttl`  

   写入后超过设定的数据，cache失效，默认35秒

2. `spring.cloud.loadbalancer.cache.capacity`
   初始容量， 默认256

3. `spring.cloud.loadbalancer.cache.enabled=false`



### 4. [Zone-Based Load-Balancing](https://docs.spring.io/spring-cloud-commons/docs/current/reference/html/#zone-based-load-balancing)

>  如果服务器集群中，比如北京的只开放支持访问天津中心的服务器组。

使用 `ZonePreferenceServiceInstanceListSupplier` 支持 zone-based load-balancing， 可以通过`DiscoveryClient` 指定`zone` 配置。也可以通过`spring.cloud.loadbalancer.zone` 配置覆盖设定。

`ZonePreferenceServiceInstanceListSupplier` 会依据`zone` 设定过滤，如果`zone` 设定null, 则获取所有的服务节点。这个对象也需要在spring context中注册

推荐的方式

```java
You could use this sample configuration to set it up:

public class CustomLoadBalancerConfiguration {

    @Bean
    public ServiceInstanceListSupplier discoveryClientServiceInstanceListSupplier(
            ConfigurableApplicationContext context) {
        return ServiceInstanceListSuppliers.builder()
                    .withDiscoveryClient()
                    .withZonePreference()
                    .withCaching()
                    .build(context);
    }
}
```



### 5. [Instance Health-Check for LoadBalancer](https://docs.spring.io/spring-cloud-commons/docs/current/reference/html/#instance-health-check-for-loadbalancer)

负载均衡需要定期检查服务器状态，使用`HealthCheckServiceInstanceListSupplier` 实现， 他可以帮助 `ServiceInstanceListSupplier` 扫描得到当前所有激活的服务器节点。

配置文件前缀`spring.cloud.loadbalancer.health-check` `initialDelay` and `interval` for the scheduler （延长时间， 多长时间检查一次）

`spring.cloud.loadbalancer.health-check.path.default`  默认连接路径，通过这个服务路径确认当前服务是否还存活

`spring.cloud.loadbalancer.health-check.path.[SERVICE_ID]` 针对不同服务器设定不同的检查服务路径

> 没有设定的话， `/actuator/health` 是默认路径。需要将`spring-boot-starter-actuator ` 加入到你的jar包依赖中

**配置**

```java
public class CustomLoadBalancerConfiguration {

    @Bean
    public ServiceInstanceListSupplier discoveryClientServiceInstanceListSupplier(
            ConfigurableApplicationContext context) {
        return ServiceInstanceListSupplier.builder()
                    .withDiscoveryClient()
                    .withHealthChecks() 	// this
                    .build(context);
        }
    }

```



### 6. [Same instance preference for LoadBalancer](https://docs.spring.io/spring-cloud-commons/docs/current/reference/html/#same-instance-preference-for-loadbalancer)

这个特性让你再次选择的时候，还是继续使用上次的节点，如果之前的节点还继续有效的情况下。

使用 `SameInstancePreferenceServiceInstanceListSupplier` , 两种方式配置，`spring.cloud.loadbalancer.configurations` 中的`same-instance-preference`, 或者 使用`ServiceInstanceListSupplier` Bean (如下)



```java
public class CustomLoadBalancerConfiguration {

    @Bean
    public ServiceInstanceListSupplier discoveryClientServiceInstanceListSupplier(
            ConfigurableApplicationContext context) {
        return ServiceInstanceListSupplier.builder()
                    .withDiscoveryClient()
                    .withSameInstancePreference() // this
                    .build(context);
        }
    }		
```



### 7. [Request-based Sticky Session for LoadBalancer](https://docs.spring.io/spring-cloud-commons/docs/current/reference/html/#request-based-sticky-session-for-loadbalancer)

可以通过cookie（提供service`instanceId`）来处理负载均衡， 当前如果将`lientRequestContext` or `ServerHttpRequestContext` 传递给LoadBalancer 

可以通过`RequestBasedStickySessionServiceInstanceListSupplier` ， 配置方式 `spring.cloud.loadbalancer.configurations` to `request-based-sticky-session` , 或者如下的配置

```java
public class CustomLoadBalancerConfiguration {

    @Bean
    public ServiceInstanceListSupplier discoveryClientServiceInstanceListSupplier(
            ConfigurableApplicationContext context) {
        return ServiceInstanceListSupplier.builder()
                    .withDiscoveryClient()
                    .withRequestBasedStickySession()  // this
                    .build(context);
        }
    }	
```

如果要在request forward之前，写入cookies , `spring.cloud.loadbalancer.sticky-session.add-service-instance-cookie` 配置设置为`true`, 默认情况下cookies名称为 `sc-lb-instance-id`, 你可以通过配置`spring.cloud.loadbalancer.instance-id-cookie-name` 修改他



### 8. [Spring Cloud LoadBalancer Hints](https://docs.spring.io/spring-cloud-commons/docs/current/reference/html/#spring-cloud-loadbalancer-hints)

spring cloud balancer允许你设置一个`hints` 信息通过`request` 对象， 后续会在`ReactiveLoadBalancer` 使用到。

可以针对所有的服务默认设置`spring.cloud.loadbalancer.hint.default`，  可以针对服务设定， `spring.cloud.loadbalancer.hint.[SERVICE_ID]` , 如果没有设定， `default` 是默认值

### 9. [Spring Cloud LoadBalancer Starter](https://docs.spring.io/spring-cloud-commons/docs/current/reference/html/#spring-cloud-loadbalancer-starter)

启动器，对于服务的包裹， 将`org.springframework.cloud:spring-cloud-starter-loadbalancer` 加入你的依赖列表中

### 10. [Passing Your Own Spring Cloud LoadBalancer Configuration](https://docs.spring.io/spring-cloud-commons/docs/current/reference/html/#custom-loadbalancer-configuration)

可以使用`@LoadBalancerClient ` , 也可以传递自己的配置文件信息

例子如下：(可以参考say hello例子, 直接加载自己的配置文件信息)

```java
@Configuration
@LoadBalancerClient(value = "stores", configuration = CustomLoadBalancerConfiguration.class)
public class MyConfiguration {

    @Bean
    @LoadBalanced
    public WebClient.Builder loadBalancedWebClientBuilder() {
        return WebClient.builder();
    }
}
```

可以传递多个配置信息文件

```java
@Configuration
@LoadBalancerClients({@LoadBalancerClient(value = "stores", configuration = StoresLoadBalancerClientConfiguration.class), @LoadBalancerClient(value = "customers", configuration = CustomersLoadBalancerClientConfiguration.class)})
public class MyConfiguration {

    @Bean
    @LoadBalanced
    public WebClient.Builder loadBalancedWebClientBuilder() {
        return WebClient.builder();
    }
}
```



### 11. [Spring Cloud LoadBalancer Lifecycle](https://docs.spring.io/spring-cloud-commons/docs/current/reference/html/#loadbalancer-lifecycle)

`LoadBalancerLifecycle` 描述了生命周期, 如下

- `onStart(Request<RC> request)`
- `onStartRequest(Request<RC> request, Response<T> lbResponse)`
- `onComplete(CompletionContext<RES, T, RC> completionContext)`

`supports(Class requestContextClass, Class responseClass, Class serverTypeClass)`  spring固有套路，处理对象在一个组中，用来判断是否处理当前的请求或者业务



### 12. [Spring Cloud LoadBalancer Statistics](https://docs.spring.io/spring-cloud-commons/docs/current/reference/html/#loadbalancer-micrometer-stats-lifecycle)

统计信息 ， 通过`MicrometerStatsLoadBalancerLifecycle ` 支持，  需要通过`spring.cloud.loadbalancer.stats.micrometer.enabled=true` 启用并且有一个`MeterRegistry` .

`MicrometerStatsLoadBalancerLifecycle `  会在 `MeterRegistry` 注册

- `loadbalancer.requests.active`: 监控当前活动请求数量
- `loadbalancer.requests.success`: 计时器， 记录负载均衡请求的执行时间
- `loadbalancer.requests.failed`:  计时器，用于测量以异常结束的任何负载平衡请求的执行时间；
- `loadbalancer.requests.discard`: 一个计数器，用于测量丢弃的负载平衡请求数，（没有通过负载均衡的请求数量）

