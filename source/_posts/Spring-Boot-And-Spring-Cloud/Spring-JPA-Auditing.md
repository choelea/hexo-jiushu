---
title: Spring JPA Data Auditing
description: 阐述如何使用Spring的JpaAuditing来方便的给auditing相关的字段赋值
---
阐述如何使用Spring的JpaAuditing来方便的给auditing相关的字段赋值
[官方文档](http://docs.spring.io/spring-data/jpa/docs/1.10.4.RELEASE/reference/html/#auditing)
[baeldung 示例](http://www.baeldung.com/database-auditing-jpa)

**注意：**
来自官方文档：
> If you have multiple implementations registered in the ApplicationContext, you can select the one to be used by explicitly setting the auditorAwareRef attribute of @EnableJpaAuditing.

示例：

```
@EnableJpaAuditing(auditorAwareRef="auditorProvider")
public class PersistenceConfig {
     
    ...
     
    @Bean
    AuditorAware<String> auditorProvider() {
        return new AuditorAwareImpl();
    }
     
    ...
     
}
```
