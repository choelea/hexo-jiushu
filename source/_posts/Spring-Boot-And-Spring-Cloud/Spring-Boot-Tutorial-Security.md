---
title: Spring Security  从单体到微服务的演进 - 单体web
description: Spring Security  从单体到微服务的演进 - 单体web
---
参考：[Spring Security and Angular JS](https://spring.io/guides/tutorials/spring-security-and-angular-js/) 
代码：[codes on github](https://github.com/choelea/spring-security-trail.git)

## 术语
**CSRF** - Cross-Site Request Forgery (跨站请求伪造)
**CORS** - Cross Origin Resource Sharing (跨域资源共享）

## 单体web app
创建spring web工程。 采用Spring CLI，（其他多种方式请自便） 
```
spring init --dependencies web,security ui
```
### OOTB security 
```
package com.example;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

@SpringBootApplication
@RestController
public class UIApplication {

	public static void main(String[] args) {
		SpringApplication.run(UIApplication.class, args);
	}

	@RequestMapping("/")
	public ResponseEntity<?> home() {
		return ResponseEntity.ok("hello");
	}
}

```

OOTB的情况，pom引入了spring security的依赖，不做任何配置。所有的请求都需要登录。

```
E:\>curl "http://localhost:8080"
{"timestamp":1478501120211,"status":401,"error":"Unauthorized","message":"Full authentication is required to access this resource","path":"/"}
```
从浏览器打开会提示输入用户名和密码，用户名是user，密码可以从console中看到：

```
Using default security password: 60fdacf5-347d-4f6f-ac7c-bc0a9725bc55
```
> **Look under the hood:** 没有任何个性化的配置的时候AuthenticationManagerConfiguration 会采用默认的配置。默认的security配置参考:SecurityProperties类。prefix = "security"。 可通过一下配置修改OOTB的用户名和密码。

```properties
security.user.name=joe
security.user.password=123456
```
### 自定义security
参考注解： @EnableWebSecurity

```java
package com.example.security;

import org.springframework.security.config.annotation.authentication.builders.AuthenticationManagerBuilder;
import org.springframework.security.config.annotation.web.builders.HttpSecurity;
import org.springframework.security.config.annotation.web.builders.WebSecurity;
import org.springframework.security.config.annotation.web.configuration.EnableWebSecurity;
import org.springframework.security.config.annotation.web.configuration.WebSecurityConfigurerAdapter;

/**
 * @author Joe
 *
 */
@EnableWebSecurity
public class ExtWebSecurityConfigurerAdapter extends WebSecurityConfigurerAdapter {


 	@Override
 	public void configure(WebSecurity web) throws Exception {
 		web.ignoring()
 		// Spring Security should completely ignore URLs starting with /resources/
 				.antMatchers("/resources/**");
 	}

 	@Override
 	protected void configure(HttpSecurity http) throws Exception {
 		http.authorizeRequests()
 				.antMatchers("/public/**").permitAll()
 				.antMatchers("/admin/**").hasRole("ADMIN")
 				.and()
 				// Possibly more configuration ...
 				.formLogin() // enable form based log in
 				// set permitAll for all URLs associated with Form Login
 				.permitAll();
 	}

 	@Override
 	protected void configure(AuthenticationManagerBuilder auth) throws Exception {
 		// enable in memory based authentication with a user named "user" and "admin"
 		auth.inMemoryAuthentication().withUser("user").password("password").roles("USER")
 				.and().withUser("admin").password("password").roles("USER", "ADMIN");
 	}

 	// Possibly more overridden methods ...
 
}
```
**添加对应URL的接口**

```
package com.example;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

@SpringBootApplication
@RestController
public class UIApplication {

	public static void main(String[] args) {
		SpringApplication.run(UIApplication.class, args);
	}

	@GetMapping("/public/hello")
	public ResponseEntity<?> home() {
		return ResponseEntity.ok("Hello Every one!");
	}
	
	@GetMapping("/admin/hello")
	public ResponseEntity<?> admin() {
		return ResponseEntity.ok("Hello Admin!");
	}
}

```
/public/hello 可以随意访问，/admin/hello 需要登录用户有admin的role。用user登录会抛出403的错误。
> 配置中加了`.formLogin().permitAll()` 未登录用户会直接重定向至OOTB的登录页面。去掉`.formLogin().permitAll()` 后直接访问/admin/hello会直接抛出403， 而非401 （*这点笔者暂时未能理解，未登录应该抛出401 才对，如果系统中加了授权需要区分这两个状态*）(什么场景不需要formLogin？用户如何登陆？满足这个场景的web app往往不是单体，可能是某个微服务，提供restful services)。
> 这里要明白一点：**一旦定制，OOTB的行为就会受影响，尽管看似你没有改**.
> 比如：不加`.formLogin().permitAll()` 就会导致没有登录提示框出现，访问需要授权的服务会直接抛出403

关于如何配置login的page等参考：[Java Configuration and Form Login](http://docs.spring.io/spring-security/site/docs/4.1.3.RELEASE/reference/htmlsingle/#authorize-requests)

### Spring Boot CORS 解决Trick：

[官方示例](https://spring.io/guides/gs/rest-service-cors/)
官方示例中只提到了 WebMvcConfigurerAdapter， 然而大部分程序都有WebSecurityConfigurerAdapter 相关的配置

```
@Configuration
@EnableWebSecurity
public class HrWebSecurityConfig extends WebSecurityConfigurerAdapter {
	
	@Override
	protected void configure(HttpSecurity http) throws Exception {
		http.csrf().disable();
		http.cors().and().authorizeRequests().antMatchers("/hr/**").authenticated();
	}
}
```
**上面代码的：http.cors() 非常重要, 这里java doc拷贝出来**
> Adds a CorsFilter to be used. If a bean by the name of corsFilter is provided, that CorsFilter is used. Else if corsConfigurationSource is defined, then that CorsConfiguration is used. Otherwise, if Spring MVC is on the classpath a HandlerMappingIntrospector is used.

从注解上可以看出来，我们需要一个corsFilter：

```
@Bean
	public CorsFilter corsFilter() {
	    final UrlBasedCorsConfigurationSource source = new UrlBasedCorsConfigurationSource();
	    final CorsConfiguration config = new CorsConfiguration();
	    config.setAllowCredentials(true);
	    config.addAllowedOrigin("*");
	    config.addAllowedHeader("*");
	    config.addAllowedMethod("OPTIONS");
	    config.addAllowedMethod("HEAD");
	    config.addAllowedMethod("GET");
	    config.addAllowedMethod("PUT");
	    config.addAllowedMethod("POST");
	    config.addAllowedMethod("DELETE");
	    config.addAllowedMethod("PATCH");
	    source.registerCorsConfiguration("/**", config);
	    return new CorsFilter(source);
	}
```


如果用到了zuul， 上面的配置也不能生效。 
[stackoverflow 这个issue](http://stackoverflow.com/questions/36042862/zuul-proxy-cors-header-contains-multiple-values-headers-repeated-twice-java-s)
(zuul 用在apigateway的时候，其他微服务不需要有CORS)
[Ticket on GitHub](https://github.com/spring-cloud/spring-cloud-netflix/issues/997)


