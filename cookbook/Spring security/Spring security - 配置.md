## Spring security Configuration

XML 配置具体查看

> https://docs.spring.io/spring-security/site/docs/5.3.4.RELEASE/reference/html5/#appendix-namespace

```
Authentication  [ɔːˌθentɪˈkeɪʃn] 身份验证，负责对于资源的授权验证
Authorization   [ˌɔːθərəˈzeɪʃn]  授权，负责权限信息相关扩展
```



### JAVA 方式
----------------------------------------------------------------------------------------------------------------------
	@EnableWebSecurity
	只用于展示，混合了大量的功能特性

#### [JAVA 配置核心]( 12.3.14. Overriding or Replacing Boot Auto Configuration)



### XML 方式

	https://docs.spring.io/spring-security/site/docs/5.4.2/reference/html5/#ns-minimal
----------------------------------------------------------------------------------------------------------------------

	最小配置: 拦截系统所有的请求, 拥有角色USER.
	<http>
	<intercept-url pattern="/**" access="hasRole('USER')" />
	<form-login />
	<logout />
	</http>


​	
### XML 元素
----------------------------------------------------------------------------------------------------------------------
	1. <http> 负责创建filterChainProxy和使用的过滤器列表
	2. <authentication-provider> 提供了
		DaoAuthenticationProviderbean
	3. <user-service> 
		InMemoryDaoImpl 内存中提供用户信息


​		
### XML 元素 -- <form-login> 
----------------------------------------------------------------------------------------------------------------------

	. always-use-default-target  
		
		如果设置为TRUE, 登陆成功之后永远访问的是 default_target_url设定， 和客户采用那种方式登陆的无关
		
		这个字段值对应的是 UsernamePasswordAuthenticationFilter.alwaysUseDefaultTargetUrl 属性
		
	. authentication-details-source-ref 
	  
		关联着 AuthenticationDetailsSource bean定义， 用于构建对象
	   
	. authentication-failure-handler-ref  
	  
	    指定一个失败失败之后的一个处理类, 实现接口 AuthenticationFailureHandler 
		
		全面接管登陆失败处理， authentication-failure-url 设定将失效
		
		UsernamePasswordAuthenticationFilter 中默认是 :SimpleUrlAuthenticationFailureHandler 
		
	. authentication-failure-url 
	   
		这个字段值对应的是 UsernamePasswordAuthenticationFilter.authenticationFailureUrl 属性
	   
		这个属性用于， 系统登陆失败之后的redriect url处理， 默认值是 /login?error
	   
	 . authentication-success-handler-ref 
	 
		这个属性指定一个bean用于登陆成功之后的逻辑。实现接口AuthenticationSuccessHandler， 默认实现 SavedRequestAwareAuthenticationSuccessHandler 
	   
	 . default_target_url
		
		这个字段值对应的是 UsernamePasswordAuthenticationFilter.defaultTargetUrl  属性
		
		如果没有设定， 默认值是 '/'
		
	 . login-page
	  
		这个URL对应这个登陆页面，
	   
		这个字段值对应的是 LoginUrlAuthenticationEntryPoint.loginFormUrl  属性, 默认值是 '/login'
		
	 . login-process-url
	    
		这个字段值对应的是 UsernamePasswordAuthenticationFilter.filterProcessesUrl   属性, 默认值是 '/login'
		
	 . password-parameter
		
		登陆页面http参数名称定义, 默认password
		
	 . username-parameter 
	  
	    登陆页面http参数名称定义, 默认password
		
	 . authentication-success-forward-url
	    
		这个字段值对应的是 UsernamePasswordAuthenticationFilter.authenticationSuccessHandler   属性,
		
	 . UsernamePasswordAuthenticationFilter 
	  
	    这个字段值对应的是 UsernamePasswordAuthenticationFilter.authenticationFailureHandler    属性,



### XML 元素 -- <intercept-url>	
	表达式URL
	https://docs.spring.io/spring-security/site/docs/5.4.2/reference/html5/#el-access-web	
----------------------------------------------------------------------------------------------------------------------
	设定系统URL权限过滤方式
	
	. Access 属性
		定义了当前元素匹配的所有URL的权限访问控制方式, ACCESS 支持表达式方式, 使用表达式方式需要，
		HTTP标签属性use-expressions设定为true.
	
	. Access 表达式支持列表
		https://docs.spring.io/spring-security/site/docs/5.4.2/reference/html5/#el-access


​		
​	
​	