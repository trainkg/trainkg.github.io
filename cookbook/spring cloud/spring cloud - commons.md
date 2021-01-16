## [Spring cloud bus](https://docs.spring.io/spring-cloud-commons/docs/current/reference/html/#spring-cloud-context-application-context-services)





### 1. spring cloud context : Application Context Service

​	对于通过spring构建应用，spring boot 有自己的观点. 配置信息的自动定位， 提供公共管理的开发服务`endpoint` 和监控服务，spring cloud 是在此之上的补充

​	由此可见, spring boot 相当于基础模块，提供构建，配置，监控等服务。 清晰的了解 spring boot 和 spring cloud的定位关系。对于内部项目的modal划分或者base line的结构构建也有参考意义。



#### 1.1 用于启动的spring context

​	spring cloud 应用启动时候会创建一个 `bootstrap` 的上下文， 是`main application`(main context)上下文的`parent context` ， 这个context 负责加载配置信息（可以来着外部文件，也可以来自加密文件）， 这两个上下文共享一个`envrionment`,  这个是可以应用于任意的spring 应用的外部配置信息集合，默认情况下，启动配置文件最优先执行，无法被本地配置覆盖

   bootstrap context 使用一种另外的方式配置main application context .  可以使用 `bootstrap.yml` 替代常规的 `application.yml` 或者 `.properties`, 将启动配置和main context 配置信息分开处理。

实例： bootstrap.yml

```yaml
spring:
  application:
    name: foo
  cloud:
    config:
      uri: ${SPRING_CONFIG_URI:http://localhost:8888}
```



如果使用`spring.application.name` 作为application context id, 需要将配置放在 `bootstrap.[properties | yml]` 中

如果需要设定profile，用于加载指定的配置，可以在`bootstrap.[properties | yml]` 中设定 `spring.profiles.active`

你可以取消bootstrap 处理通过设定 `spring.cloud.bootstrap.enabled=false` （可以在 system properties）



#### 1.2 application context 的层次结构

​	如果通过`SpringApplication` 或者`SpringApplicationBuilder` 构建spring context的时候，Bootstrap context 是作为parent context的。这个是一个特性，所以所有的子context 都可以通过他读取配置信息和 profile设定 （可以参考spring context的bean读取扫描方式），然后main context 也包含了一些额外的其他配置，对比其他未通过Spring Cloud Config 构建的系统。额外的配置属性有如下

- bootstrap  
- applicationConfig:  指的是 `bootstrap.[yml | properties]` ，用来配置bootstrap context. 针对context做了一个明显配置的划分。并且在设计的结构上良好的支持。

 context 的层级结构很大程度影响了配置文件的加载，通常情况子context会覆盖parent context的同名配置。

 `springApplicationBuilder` 可以共享一个`enviroment` 在所有的上下文中，但是不是默认值。 同级context 



#### 1.3 改变bootstrap 配置文件的路径

​	  三种方式 改变默认配置方式

- ​	`spring.cloud.bootstrap.name` 默认值是`bootstrap` 

  ​	改配置文件名称

- ​	`spring.cloud.bootstrap.location` 默认是空的

  ​	改配置文件配置

- ​	`spring.cloud.bootstrap.additional-location` 默认是空的

  ​	添加额外的配置，支持LIST

  如果存在Profile设定，加载配置文件名称策略和常规的spring boot应用是一样的

  

#### 1.4 远程配置覆盖

​	通常用于启动bootstrap context的配置都是来自于远程服务器 （因为针对服务启动来说，特性要求统一，通常会使用 一个配置服务器来管理所有的启动项配置信息， 比如spring cloud Config server）, 默认，我们不可以本地覆盖 （比较特殊，或者说不建议本地覆盖）， 可以通过`spring.cloud.config.allowOverride=true` 开放覆盖支持（本地模式不支持）

- `spring.cloud.config.overrideNone=true ` 

  可以覆盖所有

- `spring.cloud.config.overrideSystemProperties=false` 

  只可以覆盖 系统配置，命令行参数，环境变量



#### 1.5 自定义配置文件

​	通过定制`/META-INF/spring.factories` 方式。 使用key `org.springframework.cloud.bootstrap.BootstrapConfiguration` 



#### 1.6 自定义启动配置文件

​	通过`PropertySourceLocator` , 通过`spring.factories` 配置

​	例子：

```java
@Configuration
public class CustomPropertySourceLocator implements PropertySourceLocator {

    @Override
    public PropertySource<?> locate(Environment environment) {
        return new MapPropertySource("customProperty",
                Collections.<String, Object>singletonMap("property.from.sample.custom.source", "worked as intended"));
    }

}
```

`META-INF/spring.factories` 配置

```properties
org.springframework.cloud.bootstrap.BootstrapConfiguration=sample.custom.CustomPropertySourceLocator
```



#### 1.7 日志配置

可以在`bootstrap.[yaml | properties]` 添加相关的日志配置



#### 1.8 环境信息变更监听

系统监听 `EnvironmentChangeEvent` 事件， 通过`ApplicationListeners` 处理相应的逻辑， 这个事件对象中包含变化的信息KEY 列表，系统将会有两种处理

- 对于系统注解了`ConfigurationProperties` 的对象的重新绑定配置参数

  并发问题

- 重新设置log level设定 （如果修改的配置是 `logging.level.*` 相关的配置）

spring cloud config 客户端默认不会轮询环境中的变化，轮询是不推荐的（英文语法超出理解范畴）。因为是通过事件的，主要是推模式



#### 1.9 Refresh Scope

当配置文件变化时候，spring `@Bean` 会被标记在上`@RefreshScope` , 主要用于有状态bean 。举了一个例子

在系统初始化的时候，`DataSource` 已经根据初始化配置文件，创建了对应数据库的连接池， 并且在配置文件刷新的时候，连接池中有正在使用的连接。

期望这些连接一直持续到他们目前的事务完成之后再关闭，等待下一次进入获取连接的时候，将使用最新的配置文件来获取对应的连接。

有时候，可以强制在只初始化一次的bean上面添加注解`@RefreshScope` ,如果是临时的，可以使用注解`@RefreshScope ` 或者 指定 `classname`  通过配置属性

`spring.cloud.refresh.extra-refreshable`

标记为 Refresh scope bean 都是懒加载的，直到他们真正使用的时候，并且refresh scope为之前初始化好的对象提供了一个缓存，如果想在下一次方法调用的时候，强制重新初始化，必须清除缓存。

`RefreshScope`  其实是作为一个Bean在context中的，并且有一个`refreshAll` 方法，会清理所有在这个scope中的bean , 通过清除缓存（没有缓存下次就强制重新实例化了） ，同时也发布了服务 `/refresh`  ，可以刷新指定的 bean. 也可以调用`refresh(String)`方法

如果你想发布`/refresh	`服务，可以有如下配置

```yaml
management:
  endpoints:
    web:
      exposure:
        include: refresh
```



#### 1.10 encryption & decryption (加密解密)

spring cloud 针对 `envrioment` 加密的配置项有一个预处理器，规则和spring cloud service server 一样的，通过前缀 `encrypt.*` , 可以使用加密的配置 `{cipher}*`。

如果你需要使用加密特性，需要包含一个spring security RSA 在你的classpath中，maven 坐标是 `org.springframework.security:spring-security-rsa`

并且安装JCE扩展， 把对应的安全证书放在` JDK/jre/lib/security folder ` 目录下面



#### 1.11  endpoints 

spring cloud actuator application　做为管理平台，还有下面的一些服务

- `POST` to `/actuator/env` to update the `Environment` and rebind `@ConfigurationProperties` and log levels.

- `/actuator/refresh` 

  这个上面有介绍

- `/actuator/restart` 关闭 `ApplicationContext` 然后重启  (默认这个服务不开放).

- `/actuator/pause` and `/actuator/resume` 用来调用 `Lifecycle` 方法 (`stop()` and `start()` on the `ApplicationContext`).

  可以影响`applicationcontext` 的生命周期



### 2. Spring cloud commons : Common Abstractions

公共的服务， 比如服务发现，负载均衡，断路器， 所有的spring cloud clients都可以使用。 当然这些spring 也只是提供了抽象（类似JSR），具体的特性实现是依赖其他项目的

#### 2.1 注解`@enableDiscoveryClient`

查看`DiscoveryClient` 和 `ReactiveDiscoveryClient` 接口实现可以查看， `META-INF/spring.factories` , 配置`org.springframework.cloud.client.discovery.EnableDiscoveryClient`,  有下面三个实现库[Spring Cloud Netflix Eureka](https://cloud.spring.io/spring-cloud-netflix/), [Spring Cloud Consul Discovery](https://cloud.spring.io/spring-cloud-consul/), and [Spring Cloud Zookeeper Discovery](https://cloud.spring.io/spring-cloud-zookeeper/).

spring cloud Service默认提供了堵塞式和相应式的discovery client ,可以使用关闭对应的特性

- `pring.cloud.discovery.blocking.enabled=false`
- `spring.cloud.discovery.reactive.enabled=false`

也可以使用`spring.cloud.discovery.enabled=false` 完全关闭这个特性。

默认情况下， `DiscoveryClient` 是自动注册的，可以使用`@EnableDiscoveryClient` 中的配置`autoRegister=false` 关闭这个特性。



##### 2.1.1 心跳检测

心跳检查是常规服务集群中必要的特性，需要通过这个机制来检测服务是否还有效。

spring 接口定义 `DiscoveryHealthIndicator` ， 可以通过`spring.cloud.discovery.client.composite-indicator.enabled=false `  关闭混合式的`HealthIndicator` ,  一个公共的`HealthIndicator` 是自动注册的（类 `DiscoveryClientHealthIndicator`） ， 可以使用`spring.cloud.discovery.client.health-indicator.enabled=false` 关闭，可以使用 `spring.cloud.discovery.client.health-indicator.include-description=false` 关闭`DiscoveryClientHealthIndicator` description 域



##### 2.1.2 `DiscoveryClient`  排序

`DiscoveryClient` 是实现ordered接口的， 当有多个服务发现客户端的时候， 可以使用他定义优先级顺序。



##### 2.1.3 SimpleDiscoveryClient

如果没有`DiscoveryClient`  实现类在classpath的时候，`SimpleDiscoveryClient` 开始启用。

这种方式，可用的服务列表必须通过配置确定

`spring.cloud.discovery.client.simple.instances.service1[0].uri=http://s11:8080	`

`spring.cloud.discovery.client.simple.instances` 是一个公共的前缀



#### 2.2 服务注册

通过`ServiceRegistry` 方法 `register(Registration)` 注册， ``deregister(Registration)`` 取消注册， `Registration` 是一个标记接口，不做实现

注册器有如下一些实现

- `ZookeeperRegistration` used with `ZookeeperServiceRegistry`
- `EurekaRegistration` used with `EurekaServiceRegistry`
- `ConsulRegistration` used with `ConsulServiceRegistry`



##### 2.2.1 服务自动注册

默认情况下，服务注册是自动开启的，可以使用 `@EnableDiscoveryClient(autoRegister=false)` 或者 `spring.cloud.service-registry.auto-registration.enabled=false`  关闭这个特性

**自动注册事件**

当触发自动注册的时候，会发出两个事件

- `InstancePreRegisteredEvent`  在服务注册前发出
- `InstanceRegisteredEvent` 在服务注册后发出

和常规的spring事件一样，可以注册`ApplicationListener` 监控。

如果想关闭注册事件，可以通过`spring.cloud.service-registry.auto-registration.enabled=false` 关闭



##### 2.2.2 endpoint 注册

spring cloud commons 提供了 `/service-registry` 服务， 这个服务依赖`Registration` bean 在spring Application context中。

- GET  返回 `Registration`  bean的状态
- POST 用于改变 `Registration` 的状态

交互数据格式`json` 具体参数列表参考文档



#### 2.3 Spring RestTemplate as a load Balancer Client.

可以创建带有Load-balancer特性的 `RestTemplate` ,  (多服务器见load-Balancer必不可少)， 见下面例子 （也可以参考官方的say-hello例子）

```java
@Configuration
public class MyConfiguration {
	
    /**
     * 通过@LoadBalanced创建
     */
    @LoadBalanced
    @Bean
    RestTemplate restTemplate() {
        return new RestTemplate();
    }
}

public class MyClass {
    @Autowired
    private RestTemplate restTemplate;

    public String doOtherStuff() {
        String results = restTemplate.getForObject("http://stores/stores", String.class);
        return results;
    }
}
```

>  目前 `RestTemplate` 不会通过自动配置创建，需要自建



#### 2.4 Spring WebClient

我们访问服务也可以通过 `WebClient` , 也可以通过`load-balancer`  创建一个支持负载均衡的客户端。 例子如下

```java
@Configuration
public class MyConfiguration {

    @Bean
    @LoadBalanced
    public WebClient.Builder loadBalancedWebClientBuilder() {
        return WebClient.builder();
    }
}

public class MyClass {
    @Autowired
    private WebClient.Builder webClientBuilder;

    public Mono<String> doOtherStuff() {
        return webClientBuilder.build().get().uri("http://stores/stores")
                        .retrieve().bodyToMono(String.class);
    }
}
```



#### 2.5 失败重试机制

>  这个特性默认关闭

支持负载均衡的`RestTemplate` 可以配置支持失败重试。如果要激活他，需要添加 [Spring Retry](https://github.com/spring-projects/spring-retry)到你的classpath中。针对`WebTestClient` 可以通过配置

`spring.cloud.loadbalancer.retry.enabled=false`

关闭特性

For the non-reactive implementation, if you would like to implement a `BackOffPolicy` in your retries, you need to create a bean of type `LoadBalancedRetryFactory` and override the `createBackOffPolicy()` method. （`TODO`）

For the reactive implementation, you just need to enable it by setting `spring.cloud.loadbalancer.retry.backoff.enabled` to `false`.

- `spring.cloud.loadbalancer.retry.maxRetriesOnSameServiceInstance` 

  设置最大重试次数, 每个实例单独记数

- `spring.cloud.loadbalancer.retry.maxRetriesOnNextServiceInstance`

  最大试过多少次之后，要换服务节点

- `spring.cloud.loadbalancer.retry.retryableStatusCodes`

  重试失败状态码

重试需要间隔

---

> **针对  reactive implementation**
>
> - `spring.cloud.loadbalancer.retry.backoff.minBackoff` 设置最小回避时间
> - `spring.cloud.loadbalancer.retry.backoff.maxBackoff` 设置最大回避时间
> - `spring.cloud.loadbalancer.retry.backoff.jitter` 默认值0.5
> - `spring.cloud.loadbalancer.retry.avoidPreviousInstance=false`  避免选择之前已经连接过的服务节点



可以实现自己的`LoadBalancerRetryPolicy` 控制更多的细节在失败重试的时候

```java
@Configuration
public class MyConfiguration {
    @Bean
    LoadBalancedRetryFactory retryFactory() {
        return new LoadBalancedRetryFactory() {
            @Override
            public BackOffPolicy createBackOffPolicy(String service) {
                return new ExponentialBackOffPolicy();
            }
        };
    }
}
```

如果想多个监听器针对重试机制。可以参考下面的例子

```java
@Configuration
public class MyConfiguration {
    @Bean
    LoadBalancedRetryListenerFactory retryListenerFactory() {
        return new LoadBalancedRetryListenerFactory() {
            @Override
            public RetryListener[] createRetryListeners(String service) {
                return new RetryListener[]{new RetryListener() {
                    @Override
                    public <T, E extends Throwable> boolean open(RetryContext context, RetryCallback<T, E> callback) {
                        //TODO Do you business...
                        return true;
                    }

                    @Override
                     public <T, E extends Throwable> void close(RetryContext context, RetryCallback<T, E> callback, Throwable throwable) {
                        //TODO Do you business...
                    }

                    @Override
                    public <T, E extends Throwable> void onError(RetryContext context, RetryCallback<T, E> callback, Throwable throwable) {
                        //TODO Do you business...
                    }
                }};
            }
        };
    }
}
```



---



#### 2.5 多个 RESETEMPLATE 对象

```java
@Configuration
public class MyConfiguration {

    @LoadBalanced
    @Bean
    RestTemplate loadBalanced() {
        return new RestTemplate();
    }
	
    // 依赖默认注入对象    
    @Primary
    @Bean
    RestTemplate restTemplate() {
        return new RestTemplate();
    }
}

public class MyClass {
@Autowired
private RestTemplate restTemplate;

    /**
     * 使用 @LoadBalanced 标记使用支持负载均衡的 resttemplate.
     */
    @Autowired
    @LoadBalanced
    private RestTemplate loadBalanced;

    public String doOtherStuff() {
        return loadBalanced.getForObject("http://stores/stores", String.class);
    }

    public String doStuff() {
        return restTemplate.getForObject("http://example.com", String.class);
    }
}
```



#### 2.6 多WebClient

和`resttemplate` 统一的处理方式

```java
@Configuration
public class MyConfiguration {

    @LoadBalanced
    @Bean
    WebClient.Builder loadBalanced() {
        return WebClient.builder();
    }

    @Primary
    @Bean
    WebClient.Builder webClient() {
        return WebClient.builder();
    }
}

public class MyClass {
    @Autowired
    private WebClient.Builder webClientBuilder;

    @Autowired
    @LoadBalanced
    private WebClient.Builder loadBalanced;

    public Mono<String> doOtherStuff() {
        return loadBalanced.build().get().uri("http://stores/stores")
                        .retrieve().bodyToMono(String.class);
    }

    public Mono<String> doStuff() {
        return webClientBuilder.build().get().uri("http://example.com")
                        .retrieve().bodyToMono(String.class);
    }
}
```



#### 2.7  使用webflux的Webclient

 The Spring WebFlux can work with both reactive and non-reactive `WebClient` configurations, as the topics describe:

- [Spring WebFlux `WebClient` with `ReactorLoadBalancerExchangeFilterFunction`](https://docs.spring.io/spring-cloud-commons/docs/current/reference/html/#webflux-with-reactive-loadbalancer)

  ```java
  public class MyClass {
      @Autowired
      private ReactorLoadBalancerExchangeFilterFunction lbFunction;
  
      public Mono<String> doOtherStuff() {
          return WebClient.builder().baseUrl("http://stores")
              .filter(lbFunction)
              .build()
              .get()
              .uri("/stores")
              .retrieve()
              .bodyToMono(String.class);
      }
  }
  ```

  

- [[load-balancer-exchange-filter-functionload-balancer-exchange-filter-function\]](https://docs.spring.io/spring-cloud-commons/docs/current/reference/html/#load-balancer-exchange-filter-functionload-balancer-exchange-filter-function)

  ```java
  public class MyClass {
      @Autowired
      private LoadBalancerExchangeFilterFunction lbFunction;
  
      public Mono<String> doOtherStuff() {
          return WebClient.builder().baseUrl("http://stores")
              .filter(lbFunction)
              .build()
              .get()
              .uri("/stores")
              .retrieve()
              .bodyToMono(String.class);
      }
  }
  ```



#### 2.8 [Ignore Network Interfaces](https://docs.spring.io/spring-cloud-commons/docs/current/reference/html/#ignore-network-interfaces)

​	   服务注册排除， 如下一些方式

- interface

  **application.yml**

  ```yaml
  spring:
    cloud:
      inetutils:
        ignoredInterfaces:
          - docker0
          - veth.*
  ```

  

- list of regular expressions

  **bootstrap.yml**

  ```yaml
  spring:
    cloud:
      inetutils:
        preferredNetworks:
          - 192.168
          - 10.0
  ```

  

- se of only site-local addresses			

  **application.yml**

  ```yaml
  spring:
    cloud:
      inetutils:
        useOnlySiteLocalInterfaces: true
  ```

  

##### 

#####  







