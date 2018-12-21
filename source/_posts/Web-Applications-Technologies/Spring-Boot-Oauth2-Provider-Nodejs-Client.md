---
title:  Nodejs oauth2 结合 Spring Boot Oauth2 自建 Oauth2 Provider 
description: Nodejs oauth2 结合 Spring Boot Oauth2 自建 Oauth2 Provider 
...


# 参考文档
Spring Boot 官方文档 [Spring Security Oauth](http://projects.spring.io/spring-security-oauth/docs/oauth2.html#resource-server-configuration)

org.springframework.security.oauth2.config.annotation.web.configuration.AuthorizationServerSecurityConfiguration.class  line 89 
```
http
        	.authorizeRequests()
            	.antMatchers(tokenEndpointPath).fullyAuthenticated()  // 写死了。。。。
            	.antMatchers(tokenKeyPath).access(configurer.getTokenKeyAccess())
            	.antMatchers(checkTokenPath).access(configurer.getCheckTokenAccess())
```

# 运行测试
## 配置Hosts
为了更方便的区分，需要配置如下hosts：
```
127.0.0.1  auth.server.com  client.node.com  client.springboot.com
```




