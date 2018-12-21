---
title: 利用Spring Boot Oauth2 来熟悉oauth2 之 - Authorization Code Grant
description: 利用Spring Boot Oauth2 来熟悉oauth2 之 - Authorization Code Grant
---
### 参考文献

 - rfc 文档： [RFC 6749 - The OAuth 2.0 Authorization Framework](https://tools.ietf.org/html/rfc6749#section-4.1)
 - Spring boot oauth2 示例 [Tutorial · Spring Boot and OAuth2](https://spring.io/guides/tutorials/spring-boot-oauth2/)

### 代码：
https://github.com/spring-guides/tut-spring-boot-oauth2

### 流程图
从RFC文档复制过来的流程图， 来辅助我们理解oauth2 的Authorization Code Grant
![Authorization Code Grant](http://tech.jiu-shu.com/Web-Applications-Technologies/oauth2-flow.jpg)

### 探究竟
clone代码，将maven的工程（tut-spring-boot-oauth2\simple）导入eclipse跑起来。 
> You can also run all the apps on the command line using `mvn spring-boot:run` or by building the jar file and running it with mvn package and `java -jar target/*.jar` 

代码就一个文件，简单如下：

```
@SpringBootApplication
@EnableOAuth2Sso
public class SocialApplication {

	public static void main(String[] args) {
		SpringApplication.run(SocialApplication.class, args);
	}

}

```
> 接力于Spring Boot的Auto Configuration 的功能，我们不需要做任何的配置。
 
由于引入了`spring-boot-starter-security` 默认情况下，所有的请求都需求登录。
Chrome打开http://localhost:8080， 并打开开发者工具，选中'Preserve log'; 浏览器将自动重定向到/login; 由于oauth2 client的引入，请求又再次被重定向至facebook进行授权；开始了授权的流程。
> 在chrome的开发者工具中Network 标签， 按住CTRL/CMD 选中Doc和Other可以过滤掉大部分静态资源请求，方便我们来分析整个流程
![oauth2-requests](http://tech.jiu-shu.com/Web-Applications-Technologies/oauth2-requests.png)
### 角色解释
官方解释请参考：[Oauth2 Roles](https://tools.ietf.org/html/rfc6749#page-6)
**在我们这个示例中：**
 - Resource Owner： 人，也就是这个facebook 账号的所有人
 - User Agent： 这里就是Chrome 浏览器
 - Client：代表Owner去请求被保户的资源的应用（这里就是Spring Boot的Web App）
 - Resource Server： Facebook 授权能访问的资源服务（这里拿到access token后获取了用户的信息，没有访问其他的Resource Server）
 - Authorization Server： 集中授权服务器，负责验证access token

### 流程分解
#### 1. 请求资源服务首页
请求资源首页：http://localhost:8080/    Response 302 ：

```
...
Location:http://localhost:8080/login
...
Set-Cookie:JSESSIONID=8C24D287C93ECDE88C3D4BD7410668FB;path=/;HttpOnly
...
```

#### 2.  跳转至登录页面
跳转至登录：http://localhost:8080/login　　
 **Request Headers:** `SESSIONID=8C24D287C93ECDE88C3D4BD7410668FB` **Reponse Headers:** `Location:https://www.facebook.com/dialog/oauth?client_id=233668646673605&redirect_uri=http://localhost:8080/login&response_type=code&state=undAFm`
#### 3. 页面将跳转至facebook,开始Authorization Code Grant的流程, 对应流程图中的步骤(A)
 **https://www.facebook.com/dialog/oauth?client_id=233668646673605&redirect_uri=http://localhost:8080/login&response_type=code&state=undAFm**
请求的URL参数包括了用户的标识，本地生成的state以及授权成功后返回到资源服务的地址。response_type=code 标识了授权的类型是“Authorization Code Grant”
> The client includes  its client identifier, requested scope, local state, and a        redirection URI to which the authorization server will send the user-agent back once access is granted (or denied).  
 
**Facebook未检测到任何用户登录信息，重定向至facebook的登录页面。**
#### 4. 请求登录页面
#### 5. 输入账号信息后，点击登录后的post 请求
登录成功后，重定向至facebook的oauth页面进行流程的（B）步骤
#### 6. 登录成功后，进入oauth页面进入流程的(B) 
截图是第二次尝试，由于第一次的时候用户针对当前client已经授权访问基本信息后。 在第二次登录后，授权的页面就不会出现， 而是直接重定向到被授权client的redirect_uri。流程中的(C) 也就同B一起在没有和用户交互的情况下完成了。
> C步骤中，服务器回应客户端的URI，包含以下参数：
code：表示授权码，必选项。该码的有效期应该很短，通常设为10分钟，客户端只能使用该码一次，否则会被授权服务器拒绝。该码与客户端ID和重定向URI，是一一对应关系。
state：如果客户端的请求中包含这个参数，认证服务器的回应也必须一模一样包含这个参数。
#### 7. 拿着facebook授权的code进行资源服务的login请求 
/login?code=******  Spring Boot Oauth2 拿到对应的code后，1)去facebook请求获取access token; 2)获取到对应的token信息后既可以通过token进一步获取用户相关信息（这两步的api 请求是在服务器端进行），进而完成`Spring Boot的Authentication`, 成功后重定向到请求1（第一步）；同时创建了新的session。

> 从上面的步骤及截图的status code可以看出来整个oauth2的流程基本是在重定向中完成。3,4,5,6 分别与资源服务无关 （即：Spring Boot Web 应用无关）。 

Spring是如何将authentication 导向/代理给 oauth2 client的？这些都在引入的依赖帮我们自动实现了。

### 其他网络资源
[理解OAuth 2.0 - 阮一峰的网络日志](http://www.ruanyifeng.com/blog/2014/05/oauth_2_0.html)