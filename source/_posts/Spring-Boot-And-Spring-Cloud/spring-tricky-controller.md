title: Tricky Part Of Spring Controller
description: Spring 中的Controller需要注意的地方
tags:
  - Spring
  - Java
categories:
  - Web技术
date: 2018-10-08 20:36:00
---

记录Spring应用中Controller容易出问题的地方

## RequestMapping 位置

![spring-controller-requestmapping-correct.jpg](/assets/preimg/Spring-Boot-And-Spring-Cloud/spring-controller-requestmapping-correct.jpg)

非简单的path，比如： robots.txt， test.html 注解在class层，无法正常工作。

![spring-controller-requestmapping-incorrect.jpg](/assets/preimg/Spring-Boot-And-Spring-Cloud/spring-controller-requestmapping-incorrect.jpg)
