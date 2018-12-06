---
title: 利用Nexus搭建Maven私服 
description: 利用Nexus搭建Maven私服
---
阐述如果利用Nexus来快速搭建maven仓库的私有服务器
私服搭建
Docker Hub链接地址： https://hub.docker.com/r/sonatype/nexus/

docker pull sonatype/nexus
mkdir /data/nexus-data && chown -R 200 /data/nexus-data
docker run -d -p 8081:8081 --name nexus -v /data/nexus-data:/nexus-data sonatype/nexus3​

本地Maven配置
​修改Maven的全局setting.xml文件如下：

文件路径： $MAVEN_HOME/conf/setting.xml

mirrors节点加入如下内容
    <mirror>
      <id>nexus</id>
      <mirrorOf>*</mirrorOf>
      <name>Nexus</name>
      <url>http://192.168.1.80:8081/repository/maven-public/</url>
    </mirror> 

profiles节点加入如下内容
    <profile>
      <id>nexus</id>
      <!--Enable snapshots for the built in central repo to direct -->
      <!--all requests to nexus via the mirror -->
      <repositories>
        <repository>
          <id>central</id>
          <url>http://central</url>
          <releases><enabled>true</enabled></releases>
          <snapshots><enabled>true</enabled></snapshots>
        </repository>
      </repositories>
     <pluginRepositories>
        <pluginRepository>
          <id>central</id>
          <url>http://central</url>
          <releases><enabled>true</enabled></releases>
          <snapshots><enabled>true</enabled></snapshots>
        </pluginRepository>
      </pluginRepositories>
    </profile> 

 
activeProfiles​节点加入
<activeProfile>nexus</activeProfile>​


对于Snapshot的jar，如果想及时的更新，可以在maven参数中加上-U，就可以获得最新的jar包。
本地组件deploy
除了配置本地Maven配置外，还需要在setting.xml文件中加入如下内容：

servers节点
    <server>
      <id>maven-releases</id>
      <username>admin</username>
      <password>admin123</password>
    </server>
    <server>
      <id>maven-snapshots</id>
      <username>admin</username>
      <password>admin123</password>
    </server>  

项目的pom.xml文件，加入如下配置：
  <distributionManagement>
      <repository>
          <id>maven-releases</id>
          <url>http://192.168.1.​80:8081/repository/maven-releases/</url>
      </repository>
      <snapshotRepository>
          <id>maven-snapshots</id>
          <url>http://192.168.1.80:8081/repository/maven-snapshots/</url>
      </snapshotRepository>
  </distributionManagement>​