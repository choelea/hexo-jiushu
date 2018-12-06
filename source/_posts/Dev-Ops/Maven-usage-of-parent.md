---
title: Maven父子工程的搭建
description: 总结通过maven创建父子工程的方式。
---
尝试dubbo+spring的同时，总结下通过maven创建父子工程的方法。（不考虑unit test）
## 版本
**Spring Boot：** 1.4.7.RELEASE
**Maven：** 3.2.5
## 工具
eclipse
## 参考
https://github.com/dubbo/dubbo-spring-boot-project
http://blog.csdn.net/yaerfeng/article/details/26448417
http://blog.csdn.net/isea533/article/details/73744497
## maven 国内镜像
如果不翻墙，下载maven的依赖相当慢，可以添加阿里云的镜像， 速度相当快。
修改conf文件夹下的settings.xml文件，添加如下镜像配置：
```
<mirrors>
    <mirror>
      <id>alimaven</id>
      <name>aliyun maven</name>
      <url>http://maven.aliyun.com/nexus/content/groups/public/</url>
      <mirrorOf>central</mirrorOf>        
    </mirror>
  </mirrors>
```

## 步骤
### 创建父maven工程
#### 创建普通的maven工程，参考如下截图
![maven-create-parent](http://tech.jiu-shu.com/Dev-Ops/maven-create-parent.png)
#### 填写参数
![maven-create-parent-1](http://tech.jiu-shu.com/Dev-Ops/maven-create-parent-1.png)
#### 删除无用文件夹
![maven-create-parent-2](http://tech.jiu-shu.com/Dev-Ops/maven-create-parent-2.png)
#### 修改pom.xml

 1. packaging 从jar改成pom `<packaging>pom</packaging>`
 2. 添加spring-boot-starter-parent，添加dependency management。（maven的配置解释参考：http://www.blogjava.net/hellxoul/archive/2013/05/16/399345.html）
 修改后配置如下：
 

```
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
	<modelVersion>4.0.0</modelVersion>

	<groupId>com.dubboot</groupId>
	<artifactId>dubboot-example</artifactId>
	<version>0.0.1-SNAPSHOT</version>
	<packaging>pom</packaging>

	<name>dubboot-example</name>
	<url>http://maven.apache.org</url>

	<properties>
		<project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
	</properties>
	<parent>
		<groupId>org.springframework.boot</groupId>
		<artifactId>spring-boot-starter-parent</artifactId>
		<version>1.4.7.RELEASE</version> <!-- keep the version same with ${springboot.version} -->
	</parent>
	<dependencyManagement> <!-- 存在的价值只是为了方便管理版本 -->
		<dependencies>
			<dependency>
				<groupId>io.dubbo.springboot</groupId>
				<artifactId>spring-boot-starter-dubbo</artifactId>
				<version>1.0.0</version>
			</dependency>
			<dependency> 
				<groupId>junit</groupId>
				<artifactId>junit</artifactId>
				<version>3.8.1</version>
				<scope>test</scope>
			</dependency>
		</dependencies>
	</dependencyManagement>
</project>
```
### 方式一：创建子maven子工程 （dubbo 服务接口）

 - 选中父maven工程右键，新建maven module，输入相关参数即可。 -      
 - 工程导入后删除测试相关：pom.xml 的junit依赖及测试相关java文件夹。
 - pom.xml 添加 `<packaging>jar</packaging>`
### 方式二：创建子maven子工程 （Spring Boot， dubbo 服务实现）
从https://start.spring.io/ 创建, 添加依赖，JPA， Validation, Mysql 及其他依赖项（不选Spring Cloud 相关）。下载后解压至父maven工程，修改pom.xml 中的parent使其匹配父工程。

```
<parent>
	<groupId>com.dubboot</groupId>
	<artifactId>dubboot-example</artifactId>
	<version>0.0.1-SNAPSHOT</version>
</parent>
```
![Spring Initiator](http://tech.jiu-shu.com/Dev-Ops/spring-io-initiator.png)

在父工程中添加module：

```
<modules>
	<module>dubboot-jpa</module>
</modules>
```
然后可以顺利将子工程导入eclipse。

