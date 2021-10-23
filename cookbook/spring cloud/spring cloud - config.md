## [Spring Cloud Config](https://spring.io/projects/spring-cloud-config)

[TOC]





负责搭建配置中心

### 特性

Spring Cloud Config Server features:

- HTTP, resource-based API for external configuration (name-value pairs, or equivalent YAML content)
- Encrypt and decrypt property values (symmetric or asymmetric)
- Embeddable easily in a Spring Boot application using `@EnableConfigServer`

Config Client features (for Spring applications):

- Bind to the Config Server and initialize Spring `Environment` with remote property sources
- Encrypt and decrypt property values (symmetric or asymmetric)



快速开始

### 服务器端maven

```xml
<dependencies>
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-config-server</artifactId>
        <version>3.0.0</version>
    </dependency>
</dependencies>
```

ConfigServerApplication.java

```java
@Configuration
@EnableAutoConfiguration
@EnableDiscoveryClient
@EnableConfigServer
public class ConfigServerApplication {

	public static void main(String[] args) {
		SpringApplication.run(ConfigServerApplication.class, args);
	}

}
```

环境存储信息结构

三个元素， 目录结构 profile/application-[lable].properties

- application
- profile
- label



HTTP资源验证

> ##### 支持方式可以查看源码 EnvironmentController

```properties
# URL 自动汇总配置信息
/{application}/{profile}[/{label}]
返回application+(application-profile)+label目录下的（application-profile）+ label 目录下的application
# 访问指定文件，也表明了文件存储格式
/{application}-{profile}.yml
# 访问指定文件，也表明了文件存储格式
/{label}/{application}-{profile}.yml
# 访问指定文件，也表明了文件存储格式
/{application}-{profile}.properties
# 访问指定文件，也表明了文件存储格式
/{label}/{application}-{profile}.properties

```

### 配置优先级

在系统启动的时候，在CLOUD CONFIG服务中会打印出来，某个服务连接配置服务器的时候，可以获取的配置文件列表， 示例

```
2021-07-09 16:10:02.776  INFO 8308 --- [nio-8888-exec-2] o.s.c.c.s.e.NativeEnvironmentRepository  : Adding property source: classpath:/config/trunk/application.properties
```

上述表明在配置文件更目录下，找到了可以匹配的 `/config/trunk/application.properties` 作为服务启动配置文件

当找到多个配置文件的时候，存在引用优先级特性， 子目录文件会覆盖父目录文件设定，根据这个特性可以使用根目录配置`application.properties`作为公共的配置信息

具体的优先级顺序如下

```properties
1. application.properties
2. application-{profile}.properties
3. {label}/application.properties
4. {lable}/application-{profile}.properties
```



### 服务器启动时候的文件匹配规则

在文件启动的时候， 服务器会使用下面信息去查找配置

```
2021-07-09 16:30:02.818  INFO 11252 --- [           main] c.c.c.ConfigServicePropertySourceLocator : Located environment: name=task-center, profiles=[task], label=trunk, version=null, state=null
```

服务启动时候读取配置通过连接 获取配置  `http://localhost:8888/task-center/core/task`

匹配规则如下

* 根目录下applicatjion-{profile}.properties 会匹配
* label目录下，application.properties 会匹配
* lable目录下，applicatjion-{profile}.properties 会匹配

> 如果配置文件信息为空，则不会命中， 服务资源由http方式暴露，负责的文件是 `org.springframework.cloud.config.server.environment.EnvironmentController`



**发现服务方式连接**

```properties
spring.application.name=task-center
server.port=8080

# spring cloud config settings.
# label
spring.cloud.config.label=task
# profile
spring.cloud.config.profile=core
# direct connect 
# spring.cloud.config.uri=http://localhost:8888

# use discovery serve
# open discovery
spring.cloud.config.discovery.enabled=true
# CONFIG serve service id
spring.cloud.config.discovery.serviceId=SERV-configserver
#  consul
spring.cloud.consul.host=localhost
spring.cloud.consul.port=8500
spring.cloud.consul.discovery.instance-id=INS:${spring.application.name}:${server.port}
spring.cloud.consul.discovery.service-name=SERV:${spring.application.name}

```

日志信息

```
2021-07-09 16:55:05.291  INFO 12112 --- [           main] c.c.c.ConfigServicePropertySourceLocator : Fetching config from server at : http://WIN-20160425FOB:8888/
# 上面可以看出,在服务启动的时候，已经通过发现服务查找配置服务
2021-07-09 16:55:05.804  INFO 12112 --- [           main] c.c.c.ConfigServicePropertySourceLocator : Located environment: name=task-center, profiles=[core], label=task, version=null, state=null
2021-07-09 16:55:05.805  INFO 12112 --- [           main] b.c.PropertySourceBootstrapConfiguration : Located property source: [BootstrapPropertySource {name='bootstrapProperties-classpath:/config/application-core.properties'}]
2021-07-09 16:55:05.813  INFO 12112 --- [           main] com.barley.app.task.TaskApplication      : No active profile set, falling back to default profiles: default
2021-07-09 16:55:06.910  WARN 12112 --- [           main] o.m.s.mapper.ClassPathMapperScanner      : No MyBatis mapper was found in '[com.barley.**.mappers]' package. Pl
```

### 客户端连接

maven 配置

```xml
<parent>
       <groupId>org.springframework.boot</groupId>
       <artifactId>spring-boot-starter-parent</artifactId>
       <version>{spring-boot-docs-version}</version>
       <relativePath /> <!-- lookup parent from repository -->
   </parent>

<dependencyManagement>
	<dependencies>
		<dependency>
			<groupId>org.springframework.cloud</groupId>
			<artifactId>spring-cloud-dependencies</artifactId>
			<version>{spring-cloud-version}</version>
			<type>pom</type>
			<scope>import</scope>
		</dependency>
	</dependencies>
</dependencyManagement>

<dependencies>
	<dependency>
		<groupId>org.springframework.cloud</groupId>
		<artifactId>spring-cloud-starter-config</artifactId>
	</dependency>
	<dependency>
		<groupId>org.springframework.boot</groupId>
		<artifactId>spring-boot-starter-test</artifactId>
		<scope>test</scope>
	</dependency>
</dependencies>

<build>
	<plugins>
           <plugin>
               <groupId>org.springframework.boot</groupId>
               <artifactId>spring-boot-maven-plugin</artifactId>
           </plugin>
	</plugins>
</build>

```

### Spring cloud 注解处理类

`spring-cloud-config-server/META-INF/spring.fatories`

```properties
# Autoconfiguration
org.springframework.boot.autoconfigure.EnableAutoConfiguration=\
org.springframework.cloud.config.server.config.ConfigServerAutoConfiguration,\
org.springframework.cloud.config.server.config.EncryptionAutoConfiguration
```

> 本地模式解析类是`org.springframework.cloud.config.server.config.NativeRepositoryConfiguration`
>
> [systemProperties, systemEnvironment, servletContextInitParams, vcap, servletConfigInitParams, jndiProperties] 不可使用



**本地模式配置**

> 必须使用 '/' 结尾，在配合label的时候，会在原有的路径上拼接 /label进行相关查找

```properties
# spring.cloud.config.server.native.searchLocations=file:///D:/zsqworkspace/expect configuration/config-repo/
spring.cloud.config.server.native.searchLocations=classpath:/config/	
```



### 本地导入

解析方式 `StandardConfigDataLocationResolver` , 默认只支持`application`和对应的服务器配置`spring.config.name`

[^警告]: 本地模式需要关注这点，随意配置的名称可能无法生效



### [Spring Cloud Config Client](https://docs.spring.io/spring-cloud-config/docs/current/reference/html/#_spring_cloud_config_client)

spring boot 2.4版本提供了一个新的特性，通过 `spring.config.import` ， 默认绑定到配置服务器

客户端通过应用生命周期，预准备期间进行到配置服务器进行配置加载。通过`EnvironmentPostProcessorApplicationListener` 处理

客户端加载使用`spring.cloud.config.profile`,`spring.cloud.config.label` 设置配置文件坐标

