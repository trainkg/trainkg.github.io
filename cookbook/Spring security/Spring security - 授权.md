## Spring security Authentication

[TOC]

### 1. Authentication 的核心对象

----------------------------------------------------------------------------------------------------------------------
    1. SecurityContextHolder
    2. SecurityContext
    3. Authentication
     4. GrantedAuthority
     5. AuthenticationManager 
     6. ProviderManager
     7. AuthenticationProvider 
     8. Request Credentials with AuthenticationEntryPoint 
     9. AbstractAuthenticationProcessingFilter 

#### 1.1 SecurityContextHolder

<img src="https://docs.spring.io/spring-security/site/docs/5.3.4.RELEASE/reference/html5/images/servlet/authentication/architecture/securitycontextholder.png" alt="securitycontextholder" style="zoom:67%;float:left" />

SecurityContextHolder是核心对象，持有SecurityContext， 持有授权后的信息对象,  并且SpringSecurity对于授权信息对象的结构定义没有要求.

默认情况下，SecurityContextHolder是存储在ThreadLocal中的，所以的话，对于在当前执行线程中都是可以获取SecurityContext的，可以获取到当前的授权信息。

**？Spring Cloud 多服务情况应该如何处理**

多服务情况下，都是多线程的执行过程， 明显ThreadLocal的方式已经不满足, 如果是在相同的JAVA 虚拟机中的话， spring security提供了一个方案， 可以使用配置

SecurityContextHolder， 前面两个都可以作为全局配置

* SecurityContextHolder.MODE_GLOBAL
* SecurityContextHolder.MODE_INHERITABLETHREADLOCAL， 模仿THREADLOCAL的信息结构
* SecurityContextHolder.MODE_THREADLOCAL (这个是默认的配置)

> 遗留的问题，spring cloud 方式下应该由网关管理外部请求， 如果处理每个内部服务正确拿到授权信息

#### 1.2 SecurityContext

SecurityContext的持有者是 SecurityContextHolder, 包含了 [Authentication](https://docs.spring.io/spring-security/site/docs/5.3.4.RELEASE/reference/html5/#servlet-authentication-authentication) 对象，（授权信息对象）

#### 1.3 Authentication

授权信息对象，一般系统中会扩展，因为我们需要将一些项目级别的，和授权对象有关的信息需要实例化到授权信息对象信息中

这个对象主要有两个作用

* 返回当前授权用户，获取当前授权用户信息
* 提供信息给 `AuthenticationManager` 用于权限验证，在未通过验证之前，`isAuthenticated` 方法返回`false`

Authentication包含了如下信息

* Principal , 代表这用户信息，实现了`UserDetails`接口对象

* credenticals ，通常是密码信息， 当授权之后，这个信息通常会被清除，防止泄露

* authorities， 这个表示当前用户拥有的权限列表信息`GrantedAuthority`， 这个需要看权限设计，比如Profiles，roles,  permissions. role的结构也会有很大的不同

  通常在项目中会进行扩展.

#### 1.4 GrantedAuthority

上面已经提及到，这个是权限列表.

#### 1.5 AuthenticationManager

这个看名字就知道是负责整个的用户权限认证流程， 这个类的公共实现是`ProviderManager`

#### 1.6 ProviderManager

公共实现， 他是基于代理模式实现，代理列表`AuthenticationProvider`, 每个 `AuthenticationProvider` 都可以去判断授权是否成功，失败， 或者无法决定授权结果。如果当前无法作出判断，则让下一个继续判断， 如果所有的`AuthenticationProvider`都没有判定结果， 则验证失败，并且抛出异常`ProviderNotFoundException`, 流程图如下

<img src="https://docs.spring.io/spring-security/site/docs/5.3.4.RELEASE/reference/html5/images/servlet/authentication/architecture/providermanager.png" alt="providermanager" style="zoom:67%;float:left" />



实际项目中， 每个`AuthenticationProvider` 执行不同的逻辑， 验证不同的内容。

ProviderManager还允许指定自己的parent `AuthenticationManager`. 
<img src="https://docs.spring.io/spring-security/site/docs/5.3.4.RELEASE/reference/html5/images/servlet/authentication/architecture/providermanager-parent.png" alt="providermanager parent" style="zoom:67%;float:left" />











实际上，还允许多个`AuthenticationProvider` 共享一个`AuthenticationManager` , 通常情况是有多个`SecurityFilterChain` (查看spring-boot.md 介绍),

<img src="https://docs.spring.io/spring-security/site/docs/5.3.4.RELEASE/reference/html5/images/servlet/authentication/architecture/providermanagers-parent.png" alt="providermanagers parent" style="zoom:67%;float:left" />

通常情况下，ProviderManager会清除认证信息等敏感信息， 如果使用了缓存, 要多考虑一下方案

#### 1.7  AuthenticationProvider

`providerManager` 可以持有多个`AuthenticationProvider`,  不同类型不同功能，比如`DaoAuthenticationProvider` 提供了用户名密码服务。

#### 1.8  Request Credentials with `AuthenticationEntryPoint`

[`AuthenticationEntryPoint`](https://docs.spring.io/spring-security/site/docs/current/api/org/springframework/security/web/AuthenticationEntryPoint.html) 是用来获取授权信息的。比如让客户进入login页面。

#### 1.9 AbstractAuthenticationProcessingFilter

这个是一个基础`Filter`用来执行验证。在获取到客户的用户密码信息之后（credentials）， 执行验证

<img src="https://docs.spring.io/spring-security/site/docs/5.3.4.RELEASE/reference/html5/images/servlet/authentication/architecture/abstractauthenticationprocessingfilter.png" alt="abstractauthenticationprocessingfilter" style="zoom:67%;float:left" />

#### 1.10 Username/Password Authentication

这个使用广泛的验证方式,  用户名密码模式。

**获取用户名和密码**

三种从请求中获取信息的方式:

- [Form Login](https://docs.spring.io/spring-security/site/docs/5.3.4.RELEASE/reference/html5/#servlet-authentication-form)
- [Basic Authentication](https://docs.spring.io/spring-security/site/docs/5.3.4.RELEASE/reference/html5/#servlet-authentication-basic)
- [Digest Authentication](https://docs.spring.io/spring-security/site/docs/5.3.4.RELEASE/reference/html5/#servlet-authentication-digest)

**存储机制**

- 内存 [In-Memory Authentication](https://docs.spring.io/spring-security/site/docs/5.3.4.RELEASE/reference/html5/#servlet-authentication-inmemory)
- 关系型数据库 [JDBC Authentication](https://docs.spring.io/spring-security/site/docs/5.3.4.RELEASE/reference/html5/#servlet-authentication-jdbc)
- 自定义 [UserDetailsService](https://docs.spring.io/spring-security/site/docs/5.3.4.RELEASE/reference/html5/#servlet-authentication-userdetailsservice)
- LDAP 服务 [LDAP Authentication](https://docs.spring.io/spring-security/site/docs/5.3.4.RELEASE/reference/html5/#servlet-authentication-ldap)

##### 表单登录模式

spring security 提供HTML form 方式支撑，下面讲解细节， 首先关注如何执行登录流程， 首先我们需要引导客户到登录页面

![loginurlauthenticationentrypoint](https://docs.spring.io/spring-security/site/docs/5.3.4.RELEASE/reference/html5/images/servlet/authentication/unpwd/loginurlauthenticationentrypoint.png)



1. 首先当我们接受到 private 请求的时候，将其标记为未经授权的的请求

2. 框架中[`FilterSecurityInterceptor`](https://docs.spring.io/spring-security/site/docs/5.3.4.RELEASE/reference/html5/#servlet-authorization-filtersecurityinterceptor) 负责标记请求，并且抛出一个`AccessDeniedException` 异常

3. 由于当前请求用户是未经过授权的， 我们通过捕获`AccessDeniedException` 将客户引导到对应配置的登录页面，通过`AuthenticationEntryPoint` 配置。

   大部分情况，实例为[`LoginUrlAuthenticationEntryPoint`](https://docs.spring.io/spring-security/site/docs/current/api/org/springframework/security/web/authentication/LoginUrlAuthenticationEntryPoint.html).

4. 浏览器跳转到登录链接/login

5. 通过LoginController对登录页面进行渲染，比如添加一些提示信息

   

当输入用户名密码之后，将经过过滤器`UsernamePasswordAuthenticationFilter` 进行账号信息验证，验证过程如下

![usernamepasswordauthenticationfilter](https://docs.spring.io/spring-security/site/docs/5.3.4.RELEASE/reference/html5/images/servlet/authentication/unpwd/usernamepasswordauthenticationfilter.png)

1. 当filter 接受到登录请求， `UsernamePasswordAuthenticationFilter` 将会创建一个`UsernamePasswordAuthenticationToken` 对象，是一个登录信息抽象对象，提取自`httpServletRequest` ,后续对这个对象进行验证

2. 接下来， `UsernamePasswordAuthenticationToken` 将会被交给`AuthenticationManager` 

3. 如果验证失败

   * 首先SecurityContextHolder会被清空
   * 执行RememberMeServices.loginFail方法， 如果没有配置RememberMe，则没有这一步
   * `AuthenticationFailureHandler` 执行.

4.  如果验证成功

   * `SessionAuthenticationStrategy`  将会被通知有新的登录人员

   * [Authentication](https://docs.spring.io/spring-security/site/docs/5.3.4.RELEASE/reference/html5/#servlet-authentication-authentication) (人员信息) 被保存在  [SecurityContextHolder](https://docs.spring.io/spring-security/site/docs/5.3.4.RELEASE/reference/html5/#servlet-authentication-securitycontextholder).

   * 执行RememberMeServices.loginSuccess方法， 如果没有配置RememberMe，则没有这一步

   * ApplicationEventPublisher 发布一个 InteractiveAuthenticationSuccessEvent事件，用于扩展

   * AuthenticationSuccessHandler 将会被调用， SimpleUrlAuthenticationSuccessHandler 会跳转到之前用户登录的请求

     它被保存在 [`ExceptionTranslationFilter`](https://docs.spring.io/spring-security/site/docs/5.3.4.RELEASE/reference/html5/#servlet-exceptiontranslationfilter) 

通常我们是需要自定义登录页面的， 可以使用下面的配置修改，在对应项目的`WebSecurityConfigurerAdapter`实现对象中

```JAVA
protected void configure(HttpSecurity http) {
    http
        // ...
        .formLogin(withDefaults());
}

// 自定义
protected void configure(HttpSecurity http) throws Exception {
    http
        // ...
        .formLogin(form -> form
            .loginPage("/login")
            .permitAll()
        );
}

```

同时HTML的表单信息有一些要求， 默认情况下

- The form should perform a `post` to `/login`
- The form will need to include a [CSRF Token](https://docs.spring.io/spring-security/site/docs/5.3.4.RELEASE/reference/html5/#servlet-csrf) which is [automatically included](https://docs.spring.io/spring-security/site/docs/5.3.4.RELEASE/reference/html5/#servlet-csrf-include-form-auto) by Thymeleaf.
- The form should specify the username in a parameter named `username`
- The form should specify the password in a parameter named `password`
- If the HTTP parameter error is found, it indicates the user failed to provide a valid username / password
- If the HTTP parameter logout is found, it indicates the user has logged out successfully



我们自定义页面的时候，可以借助于spring MVC, 通过指定的链接，打开对应的登录页面，比如创建下面一个Bean 

```java
@Controller
class LoginController {
    @GetMapping("/login")
    String login() {
        return "login";
    }
}
```

