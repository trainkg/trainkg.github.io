## [Creating Your Own Auto-configuration](https://docs.spring.io/spring-boot/docs/current/reference/html/spring-boot-features.html#boot-features-developing-auto-configuration)

自己代码的自动注入， 拥抱自动注入和识别，提供默认支持，无需配置，快速运行



### 1. Understanding Auto-configured Beans

自动配置是通过注解`@Configuration` , 但是通常情况是和`@ConditionalOnClass ` 和 `@ConditionalOnMissingBean` 一起使用的， 他们提供了自动配置装载的逻辑条件支持。



### 2. Locating Auto-configuration Candidates

可以通过 [`spring-boot-autoconfigure`](https://github.com/spring-projects/spring-boot/tree/v2.4.2/spring-boot-project/spring-boot-autoconfigure/src/main/java/org/springframework/boot/autoconfigure) 查看源码，（[`META-INF/spring.factories`](https://github.com/spring-projects/spring-boot/tree/v2.4.2/spring-boot-project/spring-boot-autoconfigure/src/main/resources/META-INF/spring.factories) 中定义了相应的注解解析支持类）

> 不要使用自动扫描处理这些解析对象

**必须将你的配置JAVA(带有`@Configurable`的类)，放入在指定`EnableAutoConfiguration` 下面，下面是例子**

```
org.springframework.boot.autoconfigure.EnableAutoConfiguration=\
com.mycorp.libx.autoconfigure.LibXAutoConfiguration,\
com.mycorp.libx.autoconfigure.LibXWebAutoConfiguration
```

在扩展上面 可以使用  [`@AutoConfigureAfter`](https://github.com/spring-projects/spring-boot/tree/v2.4.2/spring-boot-project/spring-boot-autoconfigure/src/main/java/org/springframework/boot/autoconfigure/AutoConfigureAfter.java) or [`@AutoConfigureBefore`](https://github.com/spring-projects/spring-boot/tree/v2.4.2/spring-boot-project/spring-boot-autoconfigure/src/main/java/org/springframework/boot/autoconfigure/AutoConfigureBefore.java)  切入到生命周期中。如果提供web相关的配置，需要在`WebMvcAutoConfiguration` 后面处理（依赖关系）， 如果需要自动配置支持排序功能，需要使用`@AutoConfigureOrder` 定义各个配置的优先级， 



### 3.Condition Annotations

可以添加多个`@Conditional` 控制加载逻辑条件，`@ConditionalOnMissingBean` 只有客户未提供实现的时候，才加载默认配置。`@Conditional` 可以加在`@Configuration` 类上， 或者带有`@Bean` 方法上面

- [Class Conditions](https://docs.spring.io/spring-boot/docs/current/reference/html/spring-boot-features.html#boot-features-class-conditions)
- [Bean Conditions](https://docs.spring.io/spring-boot/docs/current/reference/html/spring-boot-features.html#boot-features-bean-conditions)
- [Property Conditions](https://docs.spring.io/spring-boot/docs/current/reference/html/spring-boot-features.html#boot-features-property-conditions)
- [Resource Conditions](https://docs.spring.io/spring-boot/docs/current/reference/html/spring-boot-features.html#boot-features-resource-conditions)
- [Web Application Conditions](https://docs.spring.io/spring-boot/docs/current/reference/html/spring-boot-features.html#boot-features-web-application-conditions)
- [SpEL Expression Conditions](https://docs.spring.io/spring-boot/docs/current/reference/html/spring-boot-features.html#boot-features-spel-conditions)



#### 3.1 Class Conditions

```java
@Configuration(proxyBeanMethods = false)
// Some conditions
public class MyAutoConfiguration {

    // Auto-configured beans

    @Configuration(proxyBeanMethods = false)
    @ConditionalOnClass(EmbeddedAcmeService.class)
    static class EmbeddedConfiguration {

        @Bean
        @ConditionalOnMissingBean
        public EmbeddedAcmeService embeddedAcmeService() { ... }

    }

}
```



#### 3.2 Bean Conditions

```java
@Configuration(proxyBeanMethods = false)
public class MyAutoConfiguration {

    @Bean
    @ConditionalOnMissingBean
    public MyService myService() { ... }

}
```



#### 3.3 Property Conditions

依赖property

#### 3.4 Resource Conditions

依赖resource

#### 3.5 Web Application Conditions

`@ConditionalOnWarDeployment` 可以识别是否是通过war形式发布

#### 3.6 SpEL Expression Conditions

依赖spring 表达式

