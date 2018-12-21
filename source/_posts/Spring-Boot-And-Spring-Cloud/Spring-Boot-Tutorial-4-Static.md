---
title: Spring Boot 开发web 应用 - 04 静态资源
description: Spring Boot 对静态资源文件的的处理方式
...

Spring Boot 对静态资源文件的的处理方式。
在[Spring Boot: 开发web 应用 - 01 创建项目](http://blog.csdn.net/choelea/article/details/73136194) 中引入的H5 的模板，对应的资源文件（css,js,images 等）放入/src/main/resources/static 下面即可直接引用到`<link rel="stylesheet" href="/css/animate.min.css">`。接下来看看Spring Boot 是怎么服务静态资源的。

# 参考文章
* [Common application properties](http://docs.spring.io/spring-boot/docs/1.5.9.RELEASE/reference/html/common-application-properties.html)
* [Developing Web Application](https://docs.spring.io/spring-boot/docs/1.5.9.RELEASE/reference/html/boot-features-developing-web-applications.html)
* [Resourcing Versioning With Spring MVC](http://www.mscharhag.com/spring/resource-versioning-with-spring-mvc)

# 代码
https://github.com/choelea/spring-boot-trail-static-content/releases/tag/version-agnostic-URLs

# 版本
* spring boot 1.5.9

## 默认行为
OOTB 的情况下，不用任何代码Spring Boot 的Web 程序已经满足了常规静态资源的需求。
![静态资源的请求头](http://tech.jiu-shu.com/Spring-Boot-And-Spring-Cloud/static-resources.jpg)
默认情况下，静态资源的response header中并没有cache 相关的设置。（expires / cache-control header.） 这种情况下，各个浏览器的默认行为有可能不一样。 （简单测试：chrome貌似会无限期缓存，firefox缓存了一个小时)
## 设定缓存时间
在`application.properties` 中添加如下配置：（单位：秒, 3600*24*365 ）

```
spring.resources.cache-period=31536000
```
> application.properties/application.yml 常见的配置项可以参考：
[Common application properties](http://docs.spring.io/spring-boot/docs/current/reference/html/common-application-properties.html)

打开Chrome强制刷新页面，打开开发者工具，查看静态资源的response header可以看到如下的内容：

```
Cache-Control:max-age=31536000
```
打开firefox的firebug也可以检查到过期时间为一年后。

## 设置访问路径（static resource path pattern）
默认情况pattern是/** 即：根路径。请求类似http://localhost:8080/test/bootstrap.min.css 和 http://localhost:8080/css/bootstrap.min.css 只要classpath 下面对应的资源文件(/static，/public, /resources, /META-INF)下面有对应目录及文件既可以访问。
> By default Spring Boot will serve static content from a directory called /static (or /public or /resources or /META-INF/resources) in the classpath or from the root of the ServletContext. 
> 可以通过spring.resources.static-locations来改变资源文件的实体文件路径; 但是实际中基本用不上，就不赘述了。

可以通过以下的配置来设定pattern，达到给所有静态资源一个公共的路径前缀。

```
spring.mvc.static-path-pattern=/static/**
```
那么不管是js css还是图片的引入路径，都需要以/static开头。`http://localhost:8080/static/css/bootstrap.min.css` (注意：这里的项目我们没有设置server.context-path，所有端口后就直接跟URI)

## 设置版本
为了提高页面的性能及用户体验，静态资源的缓存时间需要尽可能大；但是同时面临着迭代和频繁的发布。 如何让用户能得到最新的修改的内容呢？ 发布后URL 必须变，怎么变方便又效率高？

### 不可知版本号
每次内容重启后，计算文件内容生成md5值，作为新的版本号后缀。(具体的strategy类：`org.springframework.web.servlet.resource.ContentVersionStrategy`）

```
spring.resources.chain.strategy.content.enabled=true
spring.resources.chain.strategy.content.paths=/**
```
参考Spring Boot的官方指导文档 [Static Content](https://docs.spring.io/spring-boot/docs/current/reference/html/boot-features-developing-web-applications.html)
仅仅是上面的配置还不够，如何让这些静态资源加上版本的后缀？必须通过ResourceUrlProvider 来生成提供修改对应的URL；另外需要ResourceUrlEncodingFilter来解析请求。不同的view engine支持程度不一样，做法也不一样。针对freemarker来说, 添加全局的bean到model中。 

```
@ControllerAdvice
public class ResourceUrlAdvice {
 
  @Inject
  ResourceUrlProvider resourceUrlProvider;
 
  @ModelAttribute("urls")
  public ResourceUrlProvider urls() {
    return this.resourceUrlProvider;
  }
}
```
然后在模板中引入：`<link rel="stylesheet" href="${urls.getForLookupPath('/static/css/animate.min.css')}">` 生成的HTML内容变成： `<link rel="stylesheet" href="/static/css/animate.min-e8c93399a158706b3ae3a9396a9c684e.css">
` 在文件内容未修改的情况下，即使服务器重启URL也不会变。

可以参考文章： [Resourcing Versioning With Spring MVC](http://www.mscharhag.com/spring/resource-versioning-with-spring-mvc)

