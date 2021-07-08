## Spring boot- actuator

### 1. Actuator 的设计

>Actuator
>英 [ˈæktjʊeɪtə]   美 [ˈæktjuˌeɪtər]  
>激励器;致动器;执行器;执行机构;执行元件

spring boot 致力与打造微服务，自身作为构建基础， Actuator 设计用于监控spring boot 自身服务的监控检查， 监控，审计，追踪等一切和服务自身有关的服务。

暴露actuator方式可以基于HTTP或者JMX的方式

* 生命周期
* 健康检查
* 服务监控
* 资源消耗监控， 内存，CPU



### 2. [Endpoints](https://docs.spring.io/spring-boot/docs/current/reference/html/actuator.html#actuator)

* 可以控制开启关闭

当前列表

| ID                 | Description                                                  |
| :----------------- | :----------------------------------------------------------- |
| `auditevents`      | Exposes audit events information for the current application. Requires an `AuditEventRepository` bean. |
| `beans`            | Displays a complete list of all the Spring beans in your application. |
| `caches`           | Exposes available caches.                                    |
| `conditions`       | Shows the conditions that were evaluated on configuration and auto-configuration classes and the reasons why they did or did not match. |
| `configprops`      | Displays a collated list of all `@ConfigurationProperties`.  |
| `env`              | Exposes properties from Spring’s `ConfigurableEnvironment`.  |
| `flyway`           | Shows any Flyway database migrations that have been applied. Requires one or more `Flyway` beans. |
| `health`           | Shows application health information.                        |
| `httptrace`        | Displays HTTP trace information (by default, the last 100 HTTP request-response exchanges). Requires an `HttpTraceRepository` bean. |
| `info`             | Displays arbitrary application info.                         |
| `integrationgraph` | Shows the Spring Integration graph. Requires a dependency on `spring-integration-core`. |
| `loggers`          | Shows and modifies the configuration of loggers in the application. |
| `liquibase`        | Shows any Liquibase database migrations that have been applied. Requires one or more `Liquibase` beans. |
| `metrics`          | Shows ‘metrics’ information for the current application.     |
| `mappings`         | Displays a collated list of all `@RequestMapping` paths.     |
| `quartz`           | Shows information about Quartz Scheduler jobs.               |
| `scheduledtasks`   | Displays the scheduled tasks in your application.            |
| `sessions`         | Allows retrieval and deletion of user sessions from a Spring Session-backed session store. Requires a Servlet-based web application using Spring Session. |
| `shutdown`         | Lets the application be gracefully shutdown. Disabled by default. |
| `startup`          | Shows the [startup steps data](https://docs.spring.io/spring-boot/docs/current/reference/html/features.html#features.spring-application.startup-tracking) collected by the `ApplicationStartup`. Requires the `SpringApplication` to be configured with a `BufferingApplicationStartup`. |
| `threaddump`       | Performs a thread dump.                                      |



### 2.1 打开 Endpoints

默认，上述所有的endpints都是打开的，除了`shutdown` endpoint。

你可以通过设置`management.endpoint.<id>.enabled to true or false`(`id`是endpoint的id)来决定打开还是关闭一个actuator endpoint。

举个例子，要想打开`shutdown` endpoint，增加以下内容在你的`application.properties`文件中：

```properties
management.endpoint.shutdown.enabled=true
```

 或者可以全局关闭所以，定制开放endpoints.

```properties
management.endpoints.enabled-by-default=false
management.endpoint.info.enabled=true	
```

### 2.2 Endpoints 服务的暴露方式

因为包含服务的铭感信息，谨慎设定，默认情况

| ID                 | JMX  | Web  |
| :----------------- | :--- | :--- |
| `auditevents`      | Yes  | No   |
| `beans`            | Yes  | No   |
| `caches`           | Yes  | No   |
| `conditions`       | Yes  | No   |
| `configprops`      | Yes  | No   |
| `env`              | Yes  | No   |
| `flyway`           | Yes  | No   |
| `health`           | Yes  | Yes  |
| `heapdump`         | N/A  | No   |
| `httptrace`        | Yes  | No   |
| `info`             | Yes  | No   |
| `integrationgraph` | Yes  | No   |
| `jolokia`          | N/A  | No   |
| `logfile`          | N/A  | No   |
| `loggers`          | Yes  | No   |
| `liquibase`        | Yes  | No   |
| `metrics`          | Yes  | No   |
| `mappings`         | Yes  | No   |
| `prometheus`       | N/A  | No   |
| `quartz`           | Yes  | No   |
| `scheduledtasks`   | Yes  | No   |
| `sessions`         | Yes  | No   |
| `shutdown`         | Yes  | No   |
| `startup`          | Yes  | No   |
| `threaddump`       | Yes  | No   |

可以同include & exclude双向调整在JMX & WEB中暴露的endpoints.

| Property                                    | Default |
| :------------------------------------------ | :------ |
| `management.endpoints.jmx.exposure.exclude` |         |
| `management.endpoints.jmx.exposure.include` | `*`     |
| `management.endpoints.web.exposure.exclude` |         |
| `management.endpoints.web.exposure.include` |         |

> 如果是公共服务，暴露的同时一定要做安全控制

###  2.3. Securing HTTP Endpoints

使用了spring security 作为实例， 因为都是http服务，所以都是建议通过安全框架管理的endpoints的http资源安全

```java
// 将EndpointRequest.toAnyEndpoint资源加入到spring security的配置中
Configuration(proxyBeanMethods = false)
public class MySecurityConfiguration {

    @Bean
    public SecurityFilterChain securityFilterChain(HttpSecurity http) throws Exception {
        http.requestMatcher(EndpointRequest.toAnyEndpoint())
                .authorizeRequests((requests) -> requests.anyRequest().hasRole("ENDPOINT_ADMIN"));
        http.httpBasic();
        return http.build();
    }

}
```

### 2.4. Configuring Endpoints

endpoints 存在缓存机制（返回服务信息）， 可以配置缓存时间，在缓存时间内直接返回缓存数值，防止频繁访问造成服务器压力

```properties
management.endpoint.beans.cache.time-to-live=10s
```

###  2.5. Hypermedia for Actuator Web Endpoints

```properties
management.endpoints.web.discovery.enabled=false	
```

### 2.6. CORS Support

跨域访问, 以下设置设定example可以通过GET,POST方式访问网关

```properties
management.endpoints.web.cors.allowed-origins=https://example.com
management.endpoints.web.cors.allowed-methods=GET,POST	
```

推荐采用集中配置方式

> See [CorsEndpointProperties](https://github.com/spring-projects/spring-boot/tree/v2.5.2/spring-boot-project/spring-boot-actuator-autoconfigure/src/main/java/org/springframework/boot/actuate/autoconfigure/endpoint/web/CorsEndpointProperties.java) for a complete list of options. 参考做法

###  2.7. Implementing Custom Endpoints

自定义endpoints实现

使用`@Bean` 注解类`@Endpoint`， 任何方法注解为`@ReadOperation`, `@WriteOperation`,`@DeleteOperation`  将会自动通过JMX暴露， 如果在一个web应用中， 将会通过HTTP暴露服务， 

暴露方式如下描述：

```
Endpoints can be exposed over HTTP using Jersey, Spring MVC, or Spring WebFlux. If both Jersey and Spring MVC are available, Spring MVC will be used.
```

Example

```java
@ReadOperation
public CustomData getData() {
    return new CustomData("test", 5);
}
```

可以使用 `@JmxEndpoint` or `@WebEndpoint` 限制他们暴露的方式

#### 2.7.1. Receiving Input

接受参数, 复杂数据类型不支持，只支持简单数据类型

```json
{
    "name": "test",
    "counter": 42
}
```

```java
@WriteOperation
public void updateData(String name, int counter) {
    // injects "test" and 42
}
```

#####  Input Type Conversion

复杂类型的转换方式

The parameters passed to endpoint operation methods are, if necessary, automatically converted to the required type. Before calling an operation method, the input received via JMX or an HTTP request is converted to the required types using an instance of `ApplicationConversionService` as well as any `Converter` or `GenericConverter` beans qualified with `@EndpointConverter`.

#### 2.7.2. Custom Web Endpoints

自动暴露特性

Operations on an `@Endpoint`, `@WebEndpoint`, or `@EndpointWebExtension` are automatically exposed over HTTP using Jersey, Spring MVC, or Spring WebFlux. If both Jersey and Spring MVC are available, Spring MVC will be used.

##### Path

默认的路径是  `/actuator`, 

#####  HTTP method

| Operation          | HTTP method |
| :----------------- | :---------- |
| `@ReadOperation`   | `GET`       |
| `@WriteOperation`  | `POST`      |
| `@DeleteOperation` | `DELETE`    |

#####  Consumes

For a `@WriteOperation` (HTTP `POST`) that uses the request body, the consumes clause of the predicate is `application/vnd.spring-boot.actuator.v2+json, application/json`. For all other operations the consumes clause is empty.

#####  Produces

The produces clause of the predicate can be determined by the `produces` attribute of the `@DeleteOperation`, `@ReadOperation`, and `@WriteOperation` annotations. The attribute is optional. If it is not used, the produces clause is determined automatically.

If the operation method returns `void` or `Void` the produces clause is empty. If the operation method returns a `org.springframework.core.io.Resource`, the produces clause is `application/octet-stream`. For all other operations the produces clause is `application/vnd.spring-boot.actuator.v2+json, application/json`.

##### Web Endpoint Response Status

`@ReadOperation` 成功200， 无值404

`@WriteOperation` or `@DeleteOperation` 成功200， 无值204

没有参数400

##### Web Endpoint Range Requests

An HTTP range request can be used to request part of an HTTP resource. When using Spring MVC or Spring Web Flux, operations that return a `org.springframework.core.io.Resource` automatically support range requests.

#####  Web Endpoint Security

