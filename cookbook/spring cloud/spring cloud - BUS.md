## [Spring cloud commons](https://docs.spring.io/spring-cloud-commons/docs/current/reference/html/#multiple-resttemplate-objects)

[TOC]

### 1. Bus Endpoints

spring cloud bus 提供了两种服务



#### 1.1 Bus Refresh Endpoint |`/actuator/busrefresh`

​	参考commons#refresh scope部分

​	负责刷新 服务配置， 具体和`RefreshScope`有关, 会重新绑定 `@ConfigurationProperties`



#### 1.2 Bus Env Endpoint | `/actuator/busenv`

​	负责更新每个实例的环境信息



### 2. 实例地址

​	每个实例拥有自己的service Id , 这个值是通过 `spring.cloud.bus.id` 设置， 使用冒号分割.  建议的风格是 逐步具体

`App:index:id` 这种形式.

-  `App` 信息可以来自于 `vcap.application.name ` 或者 `spring.application.name` ，可以参考`loadbanace` 示例中的application.yml文件配置

-  `index` 信息可以来自于 `vcap.application.instance_index` 或者 `spring.application.index`, `local.server.port`,`server.port`

- `id`  信息可以来自于 `vcap.application.instance_id` 或者是一个随机值

  HTTP 协议终端支持 destination, 可以指定一个的实例



### 3. 实例地址匹配方式

​	destination（目标） 参数用来查找实例，使用`:` 分割， 使用 `spring PathMacher`(spring 有大量的PathMatcher实现，e.g antPathMantcher)

​	比如 `/busenv/customer:**` 可以查找所有customer名称下的instance.



### 4. Service Id 要求唯一

​    很容易理解，服务的唯一标记，必须具有唯一性。

​	在`loader balance` 的例子中，`spring.application.name` 都是 `user` , 但是`server.port` 不一样，有实例地址中可以知道，这个时候使用了`port`作为区分,如果不唯一 `spring bus` 将无法处理事件



### 5. 自定义Message Broker

  `Message Borker` 消息代理， spring cloud bus 使用 Spring cloud stream 来广播消息。 只要将对应的实现加入依赖即可

  (`spring-cloud-starter-bus-[amqp|kafka]`) , `ampp` , `kafka` 二者选其一

  Maven kafka 实例

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-bus-kafka</artifactId>
    <version>3.0.0</version>
</dependency>
```



### 6. 消息追踪

​	Bus events, 都继承`RemoteApplicationEvent` , 如果开启配置`spring.cloud.bus.trace.enabled=true` ， 那么通过spring boot `TraceRepository` 可以看到每一个事件发送到相应的所有实例

```json
{
  "timestamp": "2015-11-26T10:24:44.411+0000",
  "info": {
    "signal": "spring.cloud.bus.ack",// 接收
    "type": "RefreshRemoteApplicationEvent",
    "id": "c4d374b7-58ea-4928-a312-31984def293b",
    "origin": "stores:8081",
    "destination": "*:**"
  }
  },
  {
  "timestamp": "2015-11-26T10:24:41.864+0000",
  "info": {
    "signal": "spring.cloud.bus.sent", // 显示发送
    "type": "RefreshRemoteApplicationEvent",
    "id": "c4d374b7-58ea-4928-a312-31984def293b",
    "origin": "customers:9000",
    "destination": "*:**"
  }
  },
  {
  "timestamp": "2015-11-26T10:24:41.862+0000",
  "info": {
    "signal": "spring.cloud.bus.ack", // 接收
    "type": "RefreshRemoteApplicationEvent",
    "id": "c4d374b7-58ea-4928-a312-31984def293b",
    "origin": "customers:9000",
    "destination": "*:**"
  }
}
```

event 类型 `RefreshRemoteApplicationEvent` 从 `customers:9000` 被 `customers:9000` 和 `stores:8081` 接受到



### 7. 广播自定义的服务

​	你可以广播所有的实现了 `RemoteApplicationEvent` 的事件， 默认传输文本格式是 `json` ,  作为远程交互的常规问题就是 序列化和饭序列化，数据接受方必须知道你使用的数据格式是什么，当然也可以使用约定

​	如果你需要添加一个新的类型定义，需要进行注册，将他放入到 `org.springframework.cloud.bus.event` 子包中（应该是一个约定）

​	定义一个事件的名称，可以通过`@JsonTypeName ` 或者通过事件命名策略

> 作为发布者和消费者都需要可以访问到这些类型定义 



#### 7.1  在自己的包下面注册事件

​	如果不想使用约定的包``org.springframework.cloud.bus.event` ，可以指定一个包路径进行events类型扫描， 通过注解`@RemoteApplicationEventScan` 

通过注解指定你想扫描的包路径。

自定义事件`MyEvent`例子如下

```java
package com.acme;

public class MyEvent extends RemoteApplicationEvent {
    ...
}
```

使用下面的方式进行注册

```java
package com.acme;

@Configuration
@RemoteApplicationEventScan
public class BusConfiguration {
    ...
}
```

如果你没有指定包，他将使用自己所在的包进行扫描

当然可以明确的指出扫描路径 设立注解`@RemoteApplicationEventScan`属性 `value`,`basePackages`,`basePackageClasses`	

```java
package com.acme;

@Configuration
//@RemoteApplicationEventScan({"com.acme", "foo.bar"})
//@RemoteApplicationEventScan(basePackages = {"com.acme", "foo.bar", "fizz.buzz"})
@RemoteApplicationEventScan(basePackageClasses = BusConfiguration.class)
public class BusConfiguration {
    ...
}
```



### 8. 配置属性列表

[appendix A: Common application propertis](https://docs.spring.io/spring-cloud-bus/docs/current/reference/html/appendix.html)



​	