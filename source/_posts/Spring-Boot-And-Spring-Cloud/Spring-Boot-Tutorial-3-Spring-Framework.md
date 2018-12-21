---
title: Spring Boot 开发web 应用 - 03 Spring Framework 回顾
description: Spring Boot 开发web 应用 - 03 Spring Framework 回顾
...
## 回顾Spring Framework

[Overview of Spring Framework](http://docs.spring.io/spring/docs/current/spring-framework-reference/html/overview.html)

手绘了完整版的依赖关系。（发现问题还望大家指出）
![完整版依赖](http://tech.jiu-shu.com/Spring-Boot-And-Spring-Cloud/spring-dependency.jpg)

简化版的依赖关系。

![简化版依赖关系](http://tech.jiu-shu.com/Spring-Boot-And-Spring-Cloud/spring-dependency-complete.jpg)

结合[Overview of Spring Framework](http://docs.spring.io/spring/docs/current/spring-framework-reference/html/overview.html) 来更好的理解 Spring Framework。

### 关于依赖的理解
#### Example
module A 中有类引用了Module B 中的class。 A依赖B吗（A -> B） ?
大部分的情况是这个样子的。。。
打开两节的项目，我们可以看到spring-boot-starter-web 帮助我们引入了如下的module： 

*spring-boot-starter-web* -> *spring-boot-starter* -> *spring-boot-autoconfigure* -> *spring-boot*

可以看出spring-boot-autoconfigure只依赖于spring-boot 模块。（spring-boot 仅依赖于spring-core和spring-context）
打开spring-boot-autoconfigure-1.5.4.RELEASE-sources.jar 文件查看其中的代码： 比如：`WebMvcAutoConfiguration` 可以发现这个类引入了很多的spring-web 及其他module的class。 （从上面的推理来看spring-boot-autoconfigure并不依赖于spring-web）

#### 验证
新建一个Spring Boot的项目， 只选择JPA 一个Dependency， 通过pom.xml  的视图查看Resolved Dependencies 可以看到spring-boot-autoconfigure被引入，而spring-web没有被引入。

#### Why

类`WebMvcAutoConfiguration` 中的如下注解决定了只有web application，这个类才会被load进来。
```
@ConditionalOnWebApplication
@ConditionalOnClass({ Servlet.class, DispatcherServlet.class,
		WebMvcConfigurerAdapter.class })
```
> Conditional 相关注解不在这一篇幅深入探究。


