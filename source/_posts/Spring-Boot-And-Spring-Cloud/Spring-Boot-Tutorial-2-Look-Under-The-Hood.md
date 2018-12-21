---
title: Spring Boot 开发web 应用 - 02 探究竟
description: 探索Spring Boot的实现原理
...
进一步去了解Spring Boot的实现原理
# Look Under The Hood
### 引入了哪些jar 文件
通过pom.xml 可以看到：

```xml
<dependencies>
	<dependency>
		<groupId>org.springframework.boot</groupId>
		<artifactId>spring-boot-starter-web</artifactId>
	</dependency>
	<dependency>
		<groupId>org.springframework.boot</groupId>
		<artifactId>spring-boot-starter-freemarker</artifactId>
	</dependency>
	<dependency>
		<groupId>org.springframework.boot</groupId>
		<artifactId>spring-boot-starter-test</artifactId>
		<scope>test</scope>
	</dependency>
</dependencies>
```
> 暂时我们可以忽略掉spring-boot-starter-freemarker，这个只是引入了对freemarker的支持。

打开Dependency Hierarchy可以看看spring-boot-starter-web帮我们引入了哪些jar。
![Dependency Hierarchy](http://tech.jiu-shu.com/Spring-Boot-And-Spring-Cloud/Dependency-Hierarchy.jpg)

可以从上面看到引入了嵌入式的tomcat；在IDE里面去查询对应的jar文件，spring-boot-starter-* 的jar文件并没有任何的java代码，只是将相关依赖集合起来提供一站式服务。

> Starter POMs are a set of convenient dependency descriptors that you can include in your application. You get a one-stop-shop for all the Spring and related technology that you need, without having to hunt through sample code and copy paste loads of dependency descriptors. For example, if you want to get started using Spring and JPA for database access, just include the spring-boot-starter-data-jpa dependency in your project, and you are good to go. [Spring Boot Starter POM.](http://docs.spring.io/spring-boot/docs/current-SNAPSHOT/reference/htmlsingle/#using-boot-starter-poms)

### spring-boot-autoconfigure 
Spring-Boot的很多便利之处就在于，你只需要引入相应的依赖， 无需过多的配置，就可以按照约定研发业务功能。 比如， 你无需配置页面文件存放位置，只需要按照约定即可。 这些便利都归功于spring-boot-autoconfigure这个包。 大部分的默认配置都在这里帮忙配置好了， 此外，借助于Java 条件型的annotation 比如： @ConditionalOnClass， 可以完成动态配置，

### Java 代码
上一节中创建了项目之后，只有两个很简单的Java 类。
#### SpringbootStaticContentApplication

```
package com.choelea.springboot.springbootstaticcontent;
.............
@SpringBootApplication
public class SpringbootStaticContentApplication {

	public static void main(String[] args) {
		SpringApplication.run(SpringbootStaticContentApplication.class, args);
	}
}
```
@SpringBootApplication 只是为了方便集合了Spring Boot 常用的几个注解。（最早的Spring Boot没有这个annotation）
The @SpringBootApplication annotation is equivalent to using @Configuration, @EnableAutoConfiguration and @ComponentScan with their default attributes. Refer to :[Using the @SpringBootApplication annotation](https://docs.spring.io/spring-boot/docs/current/reference/html/using-boot-using-springbootapplication-annotation.html)

> @SpringBootApplication 的入口住类最佳实践是放在最顶级的package中；这样默认所有子package中的class都会被Spring扫到。这点可以查看@ComponentScan 的javadoc “*If specific packages are not defined, scanning will occur from the package of the class that declares this annotation.* ” 
#### HomePageController
此类只有放在@SpringBootApplication 所在类的下级包内才会被扫描生效。
```
package com.choelea.springboot.springbootstaticcontent.controller;
.................
@Controller
public class HomePageController {	
	@RequestMapping("/")
	public String home(){
		return "index";
	}
}
```

