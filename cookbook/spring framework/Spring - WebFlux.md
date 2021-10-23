##  1. Spring WebFlux

The original web framework included in the Spring Framework, Spring Web MVC, was purpose-built for the Servlet API and Servlet containers. The reactive-stack web framework, Spring WebFlux, was added later in version 5.0. It is fully non-blocking, supports [Reactive Streams](https://www.reactive-streams.org/) back pressure, and runs on such servers as **Netty, Undertow, and Servlet 3.1+** containers.

Both web frameworks mirror the names of their source modules ([spring-webmvc](https://github.com/spring-projects/spring-framework/tree/master/spring-webmvc) and [spring-webflux](https://github.com/spring-projects/spring-framework/tree/master/spring-webflux)) and co-exist side by side in the Spring Framework. Each module is optional. Applications can use one or the other module or, in some cases, both — for example, Spring MVC controllers with the reactive `WebClient`.

### 1.1. Overview

为什么要开发WebFlux项目.

主要是使用non-blocking技术去提高并发量，通过少量的线程。

> `TODO `

#### 1.1.1. Define “Reactive”

一种响应式的设计方案

#### 1.1.2. Reactive API

Reactive Streams 在程序交互上承担了重要的作用，虽然是基础架构中的API，但是不想application API那么重，他说一个轻量级的接口， 主要还是属于功能增强接口， 非常类似JAVA8的`stream` API ，但是不只是作用于集合对象，这个就是reactive Streams扩展的功能目标

Spring WebFlux 选择 [Reactor](https://github.com/reactor/reactor)作为reactive library. 提供了 `Mono` 和 `Flux`两个接口 操作数据序列。

WebFlux 需要 Reactor作为核心依赖，同时也可以使用它和其他的reactive libraries交互通过  Reactive Streams ，参考标准， WebFlux api接收一个`publisher`作为输入信息，内部类型自动处理， 返回一个 `Mono` 或者 `Flux` 作为返回对象， 如果结合其他的reactive 框架，需要自己做相关的类型转换。

#### 1.1.3. Programming Models

`Spring-web` 模块包含了reactive的基础。包含http抽象，reactive Stream 适配对于支持的服务器，以及`webhanlder` （类似Servlet API ,但是支持non-blocking）

在此之上，spring webflux 提供了两个编程模型

- [Annotated Controllers](https://docs.spring.io/spring-framework/docs/5.2.17.RELEASE/spring-framework-reference/web-reactive.html#webflux-controller): Consistent with Spring MVC and based on the same annotations from the `spring-web` module. Both Spring MVC and WebFlux controllers support reactive (Reactor and RxJava) return types, and, as a result, it is not easy to tell them apart. One notable difference is that WebFlux also supports reactive `@RequestBody` arguments.

​      注解模式

- [Functional Endpoints](https://docs.spring.io/spring-framework/docs/5.2.17.RELEASE/spring-framework-reference/web-reactive.html#webflux-fn): Lambda-based, lightweight, and functional programming model. You can think of this as a small library or a set of utilities that an application can use to route and handle requests. **The big difference with annotated controllers is that the application is in charge of request handling from start to finish versus declaring intent through annotations and being called back.**

#### 1.1.4. Applicability

Spring MVC or WebFlux?  选哪个，那个场景适用哪个？

![spring mvc and webflux venn](https://docs.spring.io/spring-framework/docs/5.2.17.RELEASE/spring-framework-reference/images/spring-mvc-and-webflux-venn.png)



> `TODO`



#### 1.1.5. Servers

Spring WebFlux is supported on Tomcat, Jetty, Servlet 3.1+ containers, as well as on non-Servlet runtimes such as Netty and Undertow.

Spring Boot has a WebFlux starter that automates these steps. **By default, the starter uses Netty**, but it is easy to switch to Tomcat, Jetty, or Undertow **by changing your Maven or Gradle dependencies**. **Spring Boot defaults to Netty**, because it is more **widely used** in the asynchronous, non-blocking space and lets a client and a server share resources.

#### 1.1.6. Performance

不是跑的更快，只是提高负载

#### 1.1.7. Concurrency Model (并发模型)

Both Spring MVC and Spring WebFlux **support annotated controllers,** but there is a **key difference** in the concurrency model and the default assumptions for blocking and threads.

**Invoking a Blocking API**

What if you do need to use a blocking library? Both Reactor and RxJava provide the `publishOn` operator to continue processing on a different thread. That means there is an easy escape hatch. Keep in mind, however, that blocking APIs are not a good fit for this concurrency model.

不推荐这么干

**Mutable State**

In Reactor and RxJava, you declare logic through operators. At runtime, a reactive pipeline is formed where data is processed sequentially, in distinct stages. A key benefit of this is that it frees applications from having to protect mutable state because application code within that pipeline is never invoked concurrently.

这段其实讲的就是NIO多路复用技术，不会相互影响，可以先了解NIO相关技术

**Threading Model**

... 

**Configuring**

...

### 1.2. Reactive Core

The `spring-web` module contains the following **foundational support** for reactive web applications:

- [HttpHandler](https://docs.spring.io/spring-framework/docs/5.2.17.RELEASE/spring-framework-reference/web-reactive.html#webflux-httphandler): Basic contract for HTTP request handling with non-blocking I/O and Reactive Streams back pressure, along with adapters for Reactor Netty, Undertow, Tomcat, Jetty, and any Servlet 3.1+ container.

  处理HTTP请求

- [`WebHandler` API](https://docs.spring.io/spring-framework/docs/5.2.17.RELEASE/spring-framework-reference/web-reactive.html#webflux-web-handler-api): Slightly higher level, general-purpose web API for request handling, on top of which concrete programming models such as annotated controllers and functional endpoints are built.

  对接HttpHandler

#### 1.2.1. `HttpHandler`

[HttpHandler](https://docs.spring.io/spring-framework/docs/5.2.17.RELEASE/javadoc-api/org/springframework/http/server/reactive/HttpHandler.html) is a **simple contract** with a single method to handle a request and a response. It is intentionally minimal, and its main and only purpose is to be a minimal abstraction over different HTTP server APIs.

The following table describes the supported server APIs:

| Server name           | Server API used                                              | Reactive Streams support                                     |
| :-------------------- | :----------------------------------------------------------- | :----------------------------------------------------------- |
| Netty                 | Netty API                                                    | [Reactor Netty](https://github.com/reactor/reactor-netty)    |
| Undertow              | Undertow API                                                 | spring-web: Undertow to Reactive Streams bridge              |
| Tomcat                | Servlet 3.1 non-blocking I/O; Tomcat API to read and write ByteBuffers vs byte[] | spring-web: Servlet 3.1 non-blocking I/O to Reactive Streams bridge |
| Jetty                 | Servlet 3.1 non-blocking I/O; Jetty API to write ByteBuffers vs byte[] | spring-web: Servlet 3.1 non-blocking I/O to Reactive Streams bridge |
| Servlet 3.1 container | Servlet 3.1 non-blocking I/O                                 | spring-web: Servlet 3.1 non-blocking I/O to Reactive Streams bridge |

The following table describes **server dependencies** (also see [supported versions](https://github.com/spring-projects/spring-framework/wiki/What's-New-in-the-Spring-Framework)):

添加对应的maven坐标到依赖中

| Server name   | Group id                | Artifact name               |
| :------------ | :---------------------- | :-------------------------- |
| Reactor Netty | io.projectreactor.netty | reactor-netty               |
| Undertow      | io.undertow             | undertow-core               |
| Tomcat        | org.apache.tomcat.embed | tomcat-embed-core           |
| Jetty         | org.eclipse.jetty       | jetty-server, jetty-servlet |

The code snippets below show using the `HttpHandler` adapters with each server API:

...

**Servlet 3.1+ Container**

To deploy as a WAR to any Servlet 3.1+ container, you can extend and include [`AbstractReactiveWebInitializer`](https://docs.spring.io/spring-framework/docs/5.2.17.RELEASE/javadoc-api/org/springframework/web/server/adapter/AbstractReactiveWebInitializer.html) in the WAR. That class wraps an `HttpHandler` with `ServletHttpHandlerAdapter` and registers that as a `Servlet`.

#### 1.2.2. `WebHandler` API

While `HttpHandler` has a simple goal to abstract the use of different HTTP servers, the `WebHandler` API aims to provide a broader set of features commonly used in web applications such as:

- User session with attributes.
- Request attributes.
- Resolved `Locale` or `Principal` for the request.
- Access to parsed and cached form data.
- Abstractions for multipart data.
- and more..

##### Special bean types

The table below lists the components that `WebHttpHandlerBuilder` can auto-detect in a Spring ApplicationContext, or that can be registered directly with it:

| Bean name                    | Bean type                    | Count | Description                                                  |
| :--------------------------- | :--------------------------- | :---- | :----------------------------------------------------------- |
| <any>                        | `WebExceptionHandler`        | 0..N  | Provide handling for exceptions from the chain of `WebFilter` instances and the target `WebHandler`. For more details, see [Exceptions](https://docs.spring.io/spring-framework/docs/5.2.17.RELEASE/spring-framework-reference/web-reactive.html#webflux-exception-handler). |
| <any>                        | `WebFilter`                  | 0..N  | Apply interception style logic to before and after the rest of the filter chain and the target `WebHandler`. For more details, see [Filters](https://docs.spring.io/spring-framework/docs/5.2.17.RELEASE/spring-framework-reference/web-reactive.html#webflux-filters). |
| `webHandler`                 | `WebHandler`                 | 1     | The handler for the request.  **类似 servlet**               |
| `webSessionManager`          | `WebSessionManager`          | 0..1  | The manager for `WebSession` instances exposed through a method on `ServerWebExchange`. `DefaultWebSessionManager` by default. |
| `serverCodecConfigurer`      | `ServerCodecConfigurer`      | 0..1  | For access to `HttpMessageReader` instances for parsing form data and multipart data that is then exposed through methods on `ServerWebExchange`. `ServerCodecConfigurer.create()` by default. |
| `localeContextResolver`      | `LocaleContextResolver`      | 0..1  | The resolver for `LocaleContext` exposed through a method on `ServerWebExchange`. `AcceptHeaderLocaleContextResolver` by default. |
| `forwardedHeaderTransformer` | `ForwardedHeaderTransformer` | 0..1  | For processing forwarded type headers, either by extracting and removing them or by removing them only. Not used by default. |

>0..1 最多只有一个， 0..N 可以有多个

##### Form Data

`ServerWebExchange` exposes the following method for accessing form data:

```java
Mono<MultiValueMap<String, String>> getFormData();
```

The `DefaultServerWebExchange` uses the configured `HttpMessageReader` to parse form data (`application/x-www-form-urlencoded`) into a `MultiValueMap`. By default, `FormHttpMessageReader` is configured for use by the `ServerCodecConfigurer` bean (see the [Web Handler API](https://docs.spring.io/spring-framework/docs/5.2.17.RELEASE/spring-framework-reference/web-reactive.html#webflux-web-handler-api)).

#####  Multipart Data

[Web MVC](https://docs.spring.io/spring-framework/docs/5.2.17.RELEASE/spring-framework-reference/web.html#mvc-multipart)

`ServerWebExchange` exposes the following method for accessing multipart data:

...

#####  Forwarded Headers



#### 1.2.3. Filters

#### 1.2.4. Exceptions

#### 1.2.5. Codecs

#### 1.2.6. Logging

### 1.3. `DispatcherHandler`

[Web MVC](https://docs.spring.io/spring-framework/docs/5.2.17.RELEASE/spring-framework-reference/web.html#mvc-servlet)

Spring WebFlux, similarly to Spring MVC, is designed around the front controller pattern, where a central `WebHandler`, the `DispatcherHandler`, provides a shared algorithm for request processing, while actual work is performed by configurable, delegate components. This model is flexible and supports diverse workflows.

#### 1.3.1. Special Bean Types

[Web MVC](https://docs.spring.io/spring-framework/docs/5.2.17.RELEASE/spring-framework-reference/web.html#mvc-servlet-special-bean-types)

The `DispatcherHandler` delegates to special beans to process requests and render the appropriate responses. **By “special beans,” we mean Spring-managed `Object` instances that implement WebFlux framework contracts.** Those usually come with built-in contracts, but you can customize their properties, extend them, or replace them.

也就是说 DispatcherHandler管理这些特殊的Bean。

The following table lists the special beans **detected** by the `DispatcherHandler`. Note that there are also some other beans detected at a lower level (see [Special bean types](https://docs.spring.io/spring-framework/docs/5.2.17.RELEASE/spring-framework-reference/web-reactive.html#webflux-web-handler-api-special-beans) in the Web Handler API).

| Bean type              | Explanation                                                  |
| :--------------------- | :----------------------------------------------------------- |
| `HandlerMapping`       | **Map a request to a handler.** The mapping is based on some criteria, the details of which vary by `HandlerMapping` implementation — annotated controllers, simple URL pattern mappings, and others.The main `HandlerMapping` implementations are `RequestMappingHandlerMapping` for `@RequestMapping` annotated methods, `RouterFunctionMapping` for functional endpoint routes, and `SimpleUrlHandlerMapping` for explicit registrations of URI path patterns and `WebHandler` instances. |
| `HandlerAdapter`       | Help the `DispatcherHandler` to invoke a handler mapped to a request regardless of how the handler is actually invoked. For example, invoking an annotated controller requires resolving annotations. The main purpose of a `HandlerAdapter` is to shield the `DispatcherHandler` from such details. |
| `HandlerResultHandler` | Process the result from the handler invocation and finalize the response. See [Result Handling](https://docs.spring.io/spring-framework/docs/5.2.17.RELEASE/spring-framework-reference/web-reactive.html#webflux-resulthandling). |

```java
// source code
public class DispatcherHandler implements WebHandler, ApplicationContextAware {

	@Nullable
	private List<HandlerMapping> handlerMappings;

	@Nullable
	private List<HandlerAdapter> handlerAdapters;

	@Nullable
	private List<HandlerResultHandler> resultHandlers;
    ...
}
```



**Reactor  Htpp Server**

> reactor.netty.http.server.HttpServerHandle

**Spring security Flux**

> org.springframework.security.web.server.WebFilterChainProxy



**处理的请求参数为** 

`ServerWebExchange`

**实现类主要有如下**

* DefaultServerWebExchange
* DelegatingServerWebExchange
* ServerWebExchangeDecorator
* SecurityContextServerWebExchange

**HandlerMapping**



#### 1.3.2. WebFlux Config

