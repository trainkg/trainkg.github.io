## [Spring Cloud Config](https://spring.io/projects/spring-cloud-config)

负责搭建配置中心

**特性**

Spring Cloud Config Server features:

- HTTP, resource-based API for external configuration (name-value pairs, or equivalent YAML content)
- Encrypt and decrypt property values (symmetric or asymmetric)
- Embeddable easily in a Spring Boot application using `@EnableConfigServer`

Config Client features (for Spring applications):

- Bind to the Config Server and initialize Spring `Environment` with remote property sources
- Encrypt and decrypt property values (symmetric or asymmetric)



快速开始

**服务器端maven**

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

HTTP资源验证

> ##### 支持方式可以查看源码 EnvironmentController

```
/{application}/{profile}[/{label}]
返回application+(application-profile)+label目录下的（application-profile）
/{application}-{profile}.yml
/{label}/{application}-{profile}.yml
/{application}-{profile}.properties
/{label}/{application}-{profile}.properties
```

**客户端连接**

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

**Spring cloud 注解处理类**

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



### 本地导入

解析方式 `StandardConfigDataLocationResolver` , 默认只支持`application`和对应的服务器配置`spring.config.name`

[^警告]: 本地模式需要关注这点，随意配置的名称可能无法生效



## [Spring Cloud Config Client](https://docs.spring.io/spring-cloud-config/docs/current/reference/html/#_spring_cloud_config_client)

spring boot 2.4版本提供了一个新的特性，通过 `spring.config.import` ， 默认绑定到配置服务器

客户端通过应用生命周期，预准备期间进行到配置服务器进行配置加载。通过`EnvironmentPostProcessorApplicationListener` 处理

客户端加载使用`spring.cloud.config.profile`,`spring.cloud.config.label` 设置配置文件坐标

