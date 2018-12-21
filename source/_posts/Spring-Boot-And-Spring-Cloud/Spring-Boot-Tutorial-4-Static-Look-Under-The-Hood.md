---
title: Spring Boot 开发web 应用 - 04 静态资源 深入探索
description: 解读原代码进一步了解Spring Boot对静态资源文件的处理
---
解读原代码进一步了解Spring Boot对静态资源文件的处理。
## 参考文章
[Developing Web Application](https://docs.spring.io/spring-boot/docs/current/reference/html/boot-features-developing-web-applications.html)
[Using Spring Boot Auto-configuration](https://docs.spring.io/spring-boot/docs/current/reference/html/using-boot-auto-configuration.html)

### Spring Boot 自动配置
Spring Boot自动配置尝试根据您添加的jar依赖关系自动配置您的Spring应用程序。大部分情况下，Spring Boot的 AutoConfiguration类会利用@ConditionalOnClass 或者其他的@ConditionalOn** 注解来有选择地配置你的应用。 
> 比如：　在目前我们都还未加入security相关的依赖；SpringBootWebSecurityConfiguration 类实际已经引入， 但是`@ConditionalOnClass({ EnableWebSecurity.class, AuthenticationEntryPoint.class })` 决定了security的依赖被引入之前SpringBootWebSecurityConfiguration 不会有任何作用。

 #### WebMvcAutoConfiguration 类
 此类作为webmvc的配置的集合点，下面的部分子类分别实现/扩展了spring-webmvc模块的类帮助SB（Spring Boot）Web 应用快速配置。
![WebMvcAutoConfiguration](http://tech.jiu-shu.com/Spring-Boot-And-Spring-Cloud/spring-boot-auto-configuratioin.jpg)

 1. WebMvcConfigurerAdapter  集合了大部分webmvc的配置；后面涉及到的时候我们会尝试定制。
 2. EnableWebMvcConfiguration @EnableWebMvc注解的替代
 
#### 上一节的配置如何生效的
**相关properties类：** WebMvcProperties	ResourceProperties

##### spring.mvc.static-path-pattern=/static/**  及spring.resources.cache-period=31536000 参考如下:
```
void org.springframework.boot.autoconfigure.web.WebMvcAutoConfiguration.WebMvcAutoConfigurationAdapter.addResourceHandlers(ResourceHandlerRegistry registry)

```

##### spring.resources.chain.* 的配置参考如下:
```
void org.springframework.boot.autoconfigure.web.WebMvcAutoConfiguration.ResourceChainResourceHandlerRegistrationCustomizer.configureResourceChain(Chain properties, ResourceChainRegistration chain)

```