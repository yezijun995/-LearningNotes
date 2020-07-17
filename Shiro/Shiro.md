# Shiro的简介与SpringBoot简单整合



## 一、Shiro基础介绍

### 1.简单介绍

Apache Shiro是一个强大且易用的Java安全框架，执行身份验证、授权、密码和会话管理。使用Shiro的易于理解API,你可以快速、轻松地获取任何应用程序，从最小的移动应用程序到最大的网络和企业应用程序。

shiro 解决了应用安全的**四要素:**

1. 认证 - 用户身份识别，常被称为用户“登录”；
2. 授权 - 访问控制；
3. 密码加密 - 保护或隐藏数据防止被偷窥；
4. 会话管理 - 每用户相关的时间敏感的状态。

### 2.Shiro三大核心

#### 1.Subject（主体）

应用代码的直接交互对象就是Subject，也就是说Shiro对外的核心API就是Subject，Subject代表了当前“用户”，这个用户不是指具体的某一个人，与当前应用交互的任何东西都是Subject，如网络爬虫，机器人等；即一个抽象概念；所有Subject都绑定到SecurityManager，与Subject的所有交互都会委托给SecurityManager；可以把Subject认为是一个门面；SecurityManager才是实际的执行者。

#### 2.SecurityManager（安全管理器）

所有与安全有关的操作都会与SecurityManager进行交互，并且SecurityManager管理者所有的Subject，可以看出它才是Shiro的核心，它负责与Shiro的其他组件进行交互，它相当于SpringMvc中的dispatcherServlet（前端控制器）的角色。

#### 3.Realm（域）

Shiro从Realm获取安全数据（用户、角色、权限），就是说SecurityManager要验证用户身份，那么它需要从Realm获取相应的用户进行比较以确定用户身份是否合法，也需要从Realm获取用户的角色与权限来判断用户是否能进行一系列操作。可以把Realm看作DataSource数据源。

### 3.Shiro相关类介绍

![](D:\笔记\Shiro\image\1.png)

|       相关类       |                             解释                             |
| :----------------: | :----------------------------------------------------------: |
|   Authentication   |       身份认证(通俗讲为登录)，验证用户是否是合法的用户       |
|   Authorization    | 授权(通俗讲为权限验证),验证某个已经认证通过的用户是否具有某个权限，用于判断用户是否能做某些事情 |
|    Cryptography    |    加密，用于保护数据的安全性，如密码加密，而不是明文存储    |
| Session Management | 会话管理。用户登录后就是一次会话，在没有退出之前，它的所有的信息都会在会话里面。 会话可以是 JavaSE环境，也可以是Web环境 |
|    Web Support     |                 Web支持，用于集成到 Web环境                  |
|      Caching       | 缓存，用户登录之后，其用户信息，拥有的角色和权限不必每次都去查，可以提高效率，如ehcache缓存 |
|    Concurrency     | 多线程并发验证，即如在一个线程中开启另一个线程，就能把权限自动传播过去 |
|    Interations     |                集成其它应用，spring、缓存框架                |
|       Run As       |   允许一个用户假装为另一个用户(如果他们允许)的身份进行访问   |
|    Remember Me     |      记住我，即一次登录后，下次再来的话，就不用重新登录      |



## 二、Spring Boot整合Shiro代码实战

1.创建SpringBoot项目

![](D:\笔记\Shiro\image\2.png)

2.pom.xml添加Shiro依赖

````xml
<!--shiro-->
<dependency>
  <groupId>org.apache.shiro</groupId>
  <artifactId>shiro-spring</artifactId>
  <version>1.5.3</version>
</dependency>
````

3.创建对应的Controller和页面

![](D:\笔记\Shiro\image\3.png)

