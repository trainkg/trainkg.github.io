![Spring Cloud下微服务权限方案](https://pica.zhimg.com/v2-32e374b99a4332ca8bd52fca9d090617_1440w.jpg?source=172ae18b)

# Spring Cloud下微服务权限方案

## 背景

从传统的单体应用转型Spring Cloud的朋友都在问我，Spring Cloud下的微服务权限怎么管？怎么设计比较合理？从大层面讲叫服务权限，往小处拆分，分别为三块：用户认证、用户权限、服务校验。

## 用户认证

传统的单体应用可能习惯了session的存在，而到了Spring cloud的微服务化后，session虽然可以采取分布式会话来解决，但终究不是上上策。开始有人推行Spring Cloud Security结合很好的OAuth2，后面为了优化OAuth 2中Access Token的存储问题，提高后端服务的可用性和扩展性，有了更好Token验证方式JWT（JSON Web Token）。这里要强调一点的是，OAuth2和JWT这两个根本没有可比性，是两个完全不同的东西。 OAuth2是一种授权框架，而JWT是一种认证协议

## OAuth2认证框架

## OAuth2中包含四个角色：

- 资源拥有者(Resource Owner)
- 资源服务器(Resource Server)
- 授权服务器(Authorization Server)
- 客户端(Client)

## OAuth2包含4种授权模式

- 授权码（认证码）模式 （Authorization code)
- 简化（隐形）模式 (Impilict
- 用户名密码模式 (Resource Owner Password Credential)
- 客户端模式 (Client Credential)

其中，OAuth2的运行流程如下图，摘自RFC 6749：

```text
+--------+                               +---------------+
|        |--(A)- Authorization Request ->|   Resource    |
|        |                               |     Owner     |
|        |<-(B)-- Authorization Grant ---|               |
|        |                               +---------------+
|        |
|        |                               +---------------+
|        |--(C)-- Authorization Grant -->| Authorization |
| Client |                               |     Server    |
|        |<-(D)----- Access Token -------|               |
|        |                               +---------------+
|        |
|        |                               +---------------+
|        |--(E)----- Access Token ------>|    Resource   |
|        |                               |     Server    |
|        |<-(F)--- Protected Resource ---|               |
+--------+                               +---------------+
```

我们在Spring Cloud OAuth2中，所有访问微服务资源的请求都在Http Header中携带Token，被访问的服务接下来再去请求授权服务器验证Token的有效性，目前这种方式，我们需要两次或者更多次的请求，所有的Token有效性校验都落在的授权服务器上，对于我们系统的水平扩展成为一个非常大的瓶颈。

## JWT认证协议

授权服务器将用户信息和授权范围序列化后放入一个JSON字符串，然后使用Base64进行编码，最终在授权服务器用私钥对这个字符串进行签名，得到一个JSON Web Token。

假设其他所有的资源服务器都将持有一个RSA公钥，当资源服务器接收到这个在Http Header中存有Token的请求，资源服务器就可以拿到这个Token，并验证它是否使用正确的私钥签名（是否经过授权服务器签名，也就是验签）。验签通过，反序列化后就拿到Toekn中包含的有效验证信息。

其中，主体运作流程图如下：

```text
+-----------+                                     +-------------+
|           |       1-Request Authorization       |             |
|           |------------------------------------>|             |
|           |     grant_type&username&password    |             |--+
|           |                                     |Authorization|  | 2-Gen
|           |                                     |Service      |  |   JWT
|           |       3-Response Authorization      |             |<-+
|           |<------------------------------------| Private Key |
|           |    access_token / refresh_token     |             |
|           |    token_type / expire_in           |             |
|  Client   |                                     +-------------+
|           |                                 
|           |                                     +-------------+
|           |       4-Request Resource            |             |
|           |-----------------------------------> |             |
|           | Authorization: bearer Access Token  |             |--+
|           |                                     | Resource    |  | 5-Verify
|           |                                     | Service     |  |  Token
|           |       6-Response Resource           |             |<-+
|           |<----------------------------------- | Public Key  |
+-----------+                                     +-------------+
```

通过上述的方式，我们可以很好地完成服务化后的用户认证。

## 用户权限

传统的单体应用的权限拦截，大家都喜欢shiro，而且用的颇为顺手。可是一旦拆分后，这权限开始分散在各个API了，shiro还好使吗？笔者在项目中，并没有用shiro。前后端分离后，交互都是token，后端的服务无状态化，前端按钮资源化，权限放哪儿管好使？

## 抽象与设计

在介绍灵活的核心设计前，先给大家普及一个入门的概念：RBAC（Role-Based Access Control，基于角色的访问控制），就是用户通过角色与权限进行关联。简单地说，一个用户拥有若干角色，每一个角色拥有若干权限。

RBAC其实是一种分析模型，主要分为：基本模型RBAC0（Core RBAC）、角色分层模型RBAC1（Hierarchal RBAC）、角色限制模型RBAC2（Constraint RBAC）和统一模型RBAC3（Combines RBAC）。

更多详情大家可以了解：[RBAC权限模型](https://link.zhihu.com/?target=http%3A//blog.csdn.net/zwk626542417/article/details/46726491)

## 核心UML



![img](https://pic2.zhimg.com/80/v2-3e3eed438a7df522714407c5436c9bc9_720w.png)

这是笔者通过多种业务场景后抽象的RBAC关系图

## 类说明

- Group

群或组，拥有一定数量权限的集合，亦可以是权限的载体。

子类：User（用户）、Role（角色）、Position（岗位）、Unit（部门），通过用户的特定构成，形成不同业务场景的群或组，而通过对群或组的父类授权，完成了用户的权限获取。

- Permission

权限，拥有一定数量资源的集成，亦可以是资源的载体。

- Resources

权限下有资源，资源的来源有：Menu（菜单）、Button（动作权限）、页面元素（按钮、tab等）、数据权限等

- Program

程序，相关权限控制的呈现载体，可以在多个菜单中挂载。

- 常见web程序基本构成



![img](https://pic1.zhimg.com/80/v2-9f156709621fdd7685db9781829e2948_720w.png)



## 模型与微服务的关系

如果把Spring Cloud服务化后的所有api接口都定义为上文的Resources，那么我们可以看到这么一个情况。

比如一个用户的增删改查，我们的页面会这么做

![img](https://pic1.zhimg.com/80/v2-ac1fa4fc0005af11c0a7aa59fe62cbbc_720w.png)



页面元素资源编码资源URI资源请求方式查询user_btn_get/api/user/{id}GET增加user_btn_add/api/userPOST编辑user_btn_edit/api/user/{id}PUT删除user_btn_del/api/user/{id}DELETE

在抽象成上述的映射关系后，我们的前后端的资源有了参照，我们对于用户组的权限授权就容易了。比如我授予一个用户增加、删除权限。在前端我们只需要检验该资源编码的有无就可以控制按钮的显示和隐藏，而在后端我们只需要统一拦截判断该用户是否具有URI和对应请求方式即可。

至于权限的统一拦截是放置在Zuul这个网关上，还是落在具体的后端服务的拦截器上（Filter、Inteceptor），都可以轻而易举地实现。不在局限于代码的侵入性。放置Zuul流程图如下：

![img](https://pic3.zhimg.com/80/v2-efbe4a2af48e48c86867a49f04759b7a_720w.png)



要是权限的统一拦截放置在Zuul上，会有一个问题，那就是后端服务安不安全，服务只需要通过注册中心，即可对其他服务进行调用。这里就涉及到后面的第三个模块，服务之间的鉴权。

## 服务之间的鉴权

因为我们都知道服务之间开源通过注册中心寻到客户端后，直接远程过程调用的。对于生产上的各个服务，一个个敏感性的接口，我们更是需要加以保护。主题的流程如下图：

![img](https://pic4.zhimg.com/80/v2-2c574cb730d4e02834e238d0f7fc40eb_720w.png)



笔者的实现方式是基于Spring Cloud的FeignClient Inteceprot（自动申请服务token、传递当前上下文）和Mvc Inteceptor（服务token校验、更新当前上下文）来实现，从而对服务的安全性做进一步保护。

结合Spring Cloud的特性后，整体流程图如下：

![img](https://pic1.zhimg.com/80/v2-467d442498d0ed41372a7567fc36a714_720w.png)



## 优化点

虽然通过上述的用户合法性检验、用户权限拦截以及服务之间的鉴权，保证了Api接口的安全性，但是其间的Http访问频率是比较高的，请求数量上来的时候，慢的问题是就会特别明显。可以考虑一定的优化策略，比如用户权限缓存、服务授权信息的派发与缓存、定时刷新服务鉴权Token等。

