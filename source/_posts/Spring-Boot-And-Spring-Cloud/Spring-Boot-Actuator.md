---
title: Spring Boot Actuator
description: Spring Boot Actuator
...
# 监控和管理 - Spring Boot Actuator

Spring Boot Actuator是spring boot的一个子集，提供的一些监控用的API。
[官方文档](http://docs.spring.io/spring-boot/docs/current/reference/htmlsingle/#production-ready)
**使用**
只需要添加一下依赖

```
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
```
**提供了以下接口**
具体接口请参考官方文档。 
> Spring Boot 1.5 后，针对添加了权限控制， 如果需要放开权限，添加配置：`management.security.enabled=false  management.context-path=/actuator` 默认是true。 参考文档：http://docs.spring.io/spring-boot/docs/current/reference/html/common-application-properties.html. 参考[官方文档](http://docs.spring.io/spring-boot/docs/current/reference/htmlsingle/#production-ready)获取最新的endpoints.

## 示例配置如下：
```
# Spring Boot 1.5 后，针对添加了权限控制， 如果需要放开权限，添加配置：management.security.enabled=false 默认是true。 
management.security.enabled=false  
management.context-path=/actuator
# to expose everything over HTTP 
management.endpoints.web.exposure.include=*
```
Actuator通过Endpionts来允许你和Spring Boot的app进行交互来监控和管理。 访问URI示例：/actuator/health

Mapping URI		|描述	| 敏感
----------	|	------ |----------------
autoconfig	| 显示一个auto-configuration的报告，该报告展示所有auto-configuration候选者及它们被应用或未被应用的原因	| true
beans	|显示一个应用中所有Spring Beans的完整列表	|true
configprops	|显示一个所有@ConfigurationProperties的整理列表|	true
dump	|执行一个线程转储|	true
env	|暴露来自Spring　ConfigurableEnvironment的属性	|true
health	|展示应用的健康信息（当使用一个未认证连接访问时显示一个简单的’status’，使用认证连接访问则显示全部信息详情）	|false
info	|显示任意的应用信息	|false
metrics|	展示当前应用的’指标’信息	|true
mappings	|显示一个所有@RequestMapping路径的整理列表	|true
shutdown	|允许应用以优雅的方式关闭（默认情况下不启用）	|true
trace	|显示trace信息（默认为最新的一些HTTP请求）	|true

**If you are using Spring MVC, the following additional endpoints can also be used:**
请参考官方文档， 不在这里赘述