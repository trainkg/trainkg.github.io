## [Spring Security - boot](https://docs.spring.io/spring-security/site/docs/5.3.4.RELEASE/reference/html5/)

[TOC]

### 1 . Spring Boot Auto Configuration

spring boot auto configuration 做了下面的事情

1. 启用spring security 默认配置，创建了一个名叫`springSecurityFilterChain` 的 Filter
2. 创建了一个名叫`UserDetailsService` 的服务
3. 注册了一个叫做`springSecurityFilterChain` 过滤器链针对每一个请求

### 2.  Spring security 核心概念

<img src="https://docs.spring.io/spring-security/site/docs/5.3.4.RELEASE/reference/html5/images/servlet/architecture/filterchain.png" alt="filterchain" style="zoom: 67%;float:left" />



总体采用过滤器链方式来处理对于请求的控制，过滤器链可以控制顺序，自定义，对于关键的登陆和退出，人员信息确认，已经提供了一些默认的支持。

#### 2.1 DelegatingFilterProxy

这个是spring framework提供的方案，提供一种方式可以连接servlet contain 和 spring framework, 因为原生的filter是无法注入BEAN的, 所以spring 针对这种需求提供了扩展支持.
<img src="https://docs.spring.io/spring-security/site/docs/5.3.4.RELEASE/reference/html5/images/servlet/architecture/delegatingfilterproxy.png" alt="delegatingfilterproxy" style="zoom:67%;" />



#### 2.2 FilterChainProxy

<img src="https://docs.spring.io/spring-security/site/docs/5.3.4.RELEASE/reference/html5/images/servlet/architecture/filterchainproxy.png" alt="filterchainproxy" style="zoom:67%;float:left" />

由上图可以看出来, FilterChainProxy 是一个Bean Filter， 并且他是一个代理模式， 代理了一个列表 SecurityFilterChain， FilterChainProxy是被 DelegatingFilterProxy包裹的（被持有，上图2.1）。

[^Bean Filter]: 指的是可以在spring 上下文中存在的Bean对象，也具有可注入的特性

#### 2.3 SecurityFilterChain

<img src="https://docs.spring.io/spring-security/site/docs/5.3.4.RELEASE/reference/html5/images/servlet/architecture/securityfilterchain.png" alt="securityfilterchain" style="zoom:67%;float:left" />

上图在2.2 基础之上补充了Filter链表部分，其中链表中元素就是 `SecurityFilter`, Spring Security 提供了很多默认重要的Filter实现。

同时作为 高级特性，还允许FilterChainProxy持有多个`SecurityFilterChain`, 但是针对一个请求，只能命中一个`SecurityFilterChain`, 由`FilterChainProxy`自身的选择策略决定.

<img src="https://docs.spring.io/spring-security/site/docs/5.3.4.RELEASE/reference/html5/images/servlet/architecture/multi-securityfilterchain.png" alt="multi securityfilterchain" style="zoom:67%;float:left" />

#### 2.3  Security Filters

默认的Security Filter

- ChannelProcessingFilter
- ConcurrentSessionFilter
- WebAsyncManagerIntegrationFilter
- SecurityContextPersistenceFilter
- HeaderWriterFilter
- CorsFilter
- CsrfFilter
- LogoutFilter
- OAuth2AuthorizationRequestRedirectFilter
- Saml2WebSsoAuthenticationRequestFilter
- X509AuthenticationFilter
- AbstractPreAuthenticatedProcessingFilter
- CasAuthenticationFilter
- OAuth2LoginAuthenticationFilter
- Saml2WebSsoAuthenticationFilter
- [`UsernamePasswordAuthenticationFilter`](https://docs.spring.io/spring-security/site/docs/5.3.4.RELEASE/reference/html5/#servlet-authentication-usernamepasswordauthenticationfilter)
- ConcurrentSessionFilter
- OpenIDAuthenticationFilter
- DefaultLoginPageGeneratingFilter
- DefaultLogoutPageGeneratingFilter
- [`DigestAuthenticationFilter`](https://docs.spring.io/spring-security/site/docs/5.3.4.RELEASE/reference/html5/#servlet-authentication-digest)
- BearerTokenAuthenticationFilter
- [`BasicAuthenticationFilter`](https://docs.spring.io/spring-security/site/docs/5.3.4.RELEASE/reference/html5/#servlet-authentication-basic)
- RequestCacheAwareFilter
- SecurityContextHolderAwareRequestFilter
- JaasApiIntegrationFilter
- RememberMeAuthenticationFilter
- AnonymousAuthenticationFilter
- OAuth2AuthorizationCodeGrantFilter
- SessionManagementFilter
- [`ExceptionTranslationFilter`](https://docs.spring.io/spring-security/site/docs/5.3.4.RELEASE/reference/html5/#servlet-exceptiontranslationfilter)
- [`FilterSecurityInterceptor`](https://docs.spring.io/spring-security/site/docs/5.3.4.RELEASE/reference/html5/#servlet-authorization-filtersecurityinterceptor)
- SwitchUserFilter

#### 2.4 Handling Security Exceptions

异常处理是每个框架都比较关心的部分。
<img src="https://docs.spring.io/spring-security/site/docs/5.3.4.RELEASE/reference/html5/images/servlet/architecture/exceptiontranslationfilter.png" alt="exceptiontranslationfilter" style="zoom:67%;float:left;" />























首先Spring security 异常处理是基于[`ExceptionTranslationFilter`](https://docs.spring.io/spring-security/site/docs/current/api/org/springframework/security/web/access/ExceptionTranslationFilter.html)完成的.

- ![number 1](https://docs.spring.io/spring-security/site/docs/5.3.4.RELEASE/reference/html5/images/icons/number_1.png) 首先, `ExceptionTranslationFilter` 调用 `FilterChain.doFilter(request, response)` 去完成整个请求（Filter Chain 标准动作).
- ![number 2](https://docs.spring.io/spring-security/site/docs/5.3.4.RELEASE/reference/html5/images/icons/number_2.png) 如果用户未授权或者出现异常 `AuthenticationException`, 开始验证权限流程
  - 清理 [SecurityContextHolder](https://docs.spring.io/spring-security/site/docs/5.3.4.RELEASE/reference/html5/#servlet-authentication-securitycontextholder) 
  - `HttpServletRequest` 保留在 [`RequestCache`](https://docs.spring.io/spring-security/site/docs/current/api/org/springframework/security/web/savedrequest/RequestCache.html). 如果授权成功, the `RequestCache`会用来响应之前的请求
  - `AuthenticationEntryPoint` is used to request credentials from the client. For example, it might redirect to a log in page or send a `WWW-Authenticate` header.
- ![number 3](https://docs.spring.io/spring-security/site/docs/5.3.4.RELEASE/reference/html5/images/icons/number_3.png) 如果出现异常 `AccessDeniedException`, 拒绝访问. 异常 `AccessDeniedHandler` 执行后续内容.

[^credentials]:([krəˈdenʃlz], 资格证书)



