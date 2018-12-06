---
title: 使用ELK来做日志归总
description:  阐述使用ELK来做日志归总
...

# ELK 初探
ELK实时日志分析平台 初次尝试。 ELK 的多种架构请参考文章: [漫谈ELK在大数据运维中的应用](https://blog.csdn.net/lively1982/article/details/50678657)
## 平台
* CentOS 7 
* Oracle JDK 8
* Kibana 4.5.2
* Elaticsearch 2.3.4
* logstash 2.3.4
* filebeat 1.2.3
查看version command： `filebeat --version`
## 系统架构图
![elk](http://tech.jiu-shu.com/Elastic-Technologies/elk.png)
## 软件的安装
采用yum的安装模式。首先需要添加对应的repo文件。 对应的详细的安装方法可以参考在线文档， 这里以logstash为例。
### logstash 安装
- **Download and install the public signing key**

```
rpm --import https://packages.elastic.co/GPG-KEY-elasticsearch
```
- **添加Repo到目录/etc/yum.repos.d/， 比如：logstash.repo**

```
[logstash-2.3]
name=Logstash repository for 2.3.x packages
baseurl=https://packages.elastic.co/logstash/2.3/centos
gpgcheck=1
gpgkey=https://packages.elastic.co/GPG-KEY-elasticsearch
enabled=1
```
- **安装**

```
yum install logstash
```
- **随系统自动启动**
```
sudo chkconfig --add filebeat
```
### 其他软件的repositories
**filebeat**

```
[beats]
name=Elastic Beats Repository
baseurl=https://packages.elastic.co/beats/yum/el/$basearch
enabled=1
gpgkey=https://packages.elastic.co/GPG-KEY-elasticsearch
gpgcheck=1
```
**elasticsearch [官方介绍](https://www.elastic.co/guide/en/elasticsearch/reference/current/setup-repositories.html)**

```
[elasticsearch-2.x]
name=Elasticsearch repository for 2.x packages
baseurl=https://packages.elastic.co/elasticsearch/2.x/centos
gpgcheck=1
gpgkey=https://packages.elastic.co/GPG-KEY-elasticsearch
enabled=1
```
**kibana [在线文档](https://www.elastic.co/guide/en/kibana/current/setup.html)**

```
[kibana-4.5]
name=Kibana repository for 4.5.x packages
baseurl=http://packages.elastic.co/kibana/4.5/centos
gpgcheck=1
gpgkey=http://packages.elastic.co/GPG-KEY-elasticsearch
enabled=1
```
**查看服务状态**

```
servie logstash status
```

**查看服务文件路径**

```
rpm -ql logstash
```
## FileBeat 使用
filebeat 安装后的配置文件存放于：/etc/filebeat/下
修改配置文件filebeat.yml
**1， 修改文件的路径：比如：/home/osboxes/app.log**
```
filebeat:
  prospectors:
    -
      paths:
        - "/home/osboxes/app.log"
```
**2， 修改输出， 默认是直接输出到Elasticsearch，我们修改输出到logstash**
只需要打开对应的注释即可，将elasticsearch相关注释掉， 打开logstash的注释。

```
output:
  logstash:
    hosts: ["127.0.0.1:5044"]

    # Optional load balance the events between the Logstash hosts
    #loadbalance: true
```
filebeat.yml 已经配置了多个output选项，我们只需要打开注解。 这里可以做个小的测试。 修改配置后可运行命令验证：`filebeat -configtest -e.` **filebeat只能配置一个output项，修改配置后需要重启**
1，找到Console output，打开注解

```
##Console output
   console:
    # Pretty print json event
    pretty: true
```
2， 停止filebeat服务 `sudo service filebeat stop`，手动启动filebeat来方便我们观察console输出`sudo filebeat -e -c /etc/filebeat/filebeat.yml`。(On windows: `filebeat.exe -e -c filebeat.yml`)
3， 新开窗口输出信息至文件/var/log/app.log
```
echo "2016-06-29 17:14:13.802  INFO 6244 --- [main] org.hibernate.Version                    : HHH000412: Hibernate Core {4.3.11.Final}" >> app.log
```
4，切换至filebeat的启动窗口可以看到如下的输出。

```
[osboxes@osboxes logstash]$ sudo filebeat -e -c /etc/filebeat/filebeat.yml
{
  "@timestamp": "2016-07-11T13:44:43.926Z",
  "beat": {
    "hostname": "osboxes",
    "name": "osboxes"
  },
  "count": 1,
  "fields": null,
  "input_type": "log",
  "message": "2016-06-29 17:14:13.802  INFO 6244 --- [main] org.hibernate.Version                    : HHH000412: Hibernate Core {4.3.11.Final}",
  "offset": 130,
  "source": "/home/osboxes/app.log",
  "type": "log"
}

```
## LogStash 配置
 上面的小测做完后，将filebeat的配置改回输出到logstash。
### 连通filebeat和logstash
 **1， 添加logstash.conf 文件在/etc/logstash/conf.d/logstash.conf**

```
input {
  beats {
    port => 5044
  }
}

output {
  stdout{}
}
```

修改后可以通过命令验证配置是否正确：

```
sudo /opt/logstash/bin/logstash -f /etc/logstash/conf.d/logstash.conf --configtest
```

**2, 启动logstash**
采用命令启动方便从console观察输出。`sudo /opt/logstash/bin/logstash -f /etc/logstash/conf.d/logstash.conf`如果采用service的启动方式，需要去/var/log/logstash/logstash.stdout 查看log
**3，启动filebeat 然后向文件app.log 写入log**

```
echo "2016-06-29 17:14:13.802  INFO 6244 --- [main] org.hibernate.Version                    : HHH000412: Hibernate Core {4.3.11.Final}" >> app.log
```
**4，切换至logstash窗口， 可以观察到一下输出，证明filebeat已经可以成功输出到logstash**

```
[osboxes@osboxes bin]$ sudo ./logstash -f /etc/logstash/conf.d/logstash.conf 
Settings: Default pipeline workers: 1
Pipeline main started
2016-07-12T05:57:46.877Z osboxes 2016-06-29 17:14:13.802  INFO 6244 --- [main] org.hibernate.Version                    : HHH000412: Hibernate Core {4.3.11.Final}

```
### 使用Grok Filter Plugin解析日志 （spring boot 的默认日志格式）
**1， 修改logstash.conf 添加filter，重启logstash**

```
input {
  beats {
    port => 5044
  }
}
filter {
  #If log line contains tab character followed by 'at' then we will tag that entry as stacktrace
  if [message] =~ "\tat" {
    grok {
      match => ["message", "^(\tat)"]
      add_tag => ["stacktrace"]
    }
  }

  #Grokking Spring Boot's default log format
  grok {
    match => [ "message",
               "(?<timestamp>%{YEAR}-%{MONTHNUM}-%{MONTHDAY} %{TIME})  %{LOGLEVEL:level} %{NUMBER:pid} --- \[(?<thread>[A-Za-z0-9-]+)\] (?<class>[A-Za-z0-9.#_]+)\s*:\s+(?<logmessage>.*)",
               "message",
               "(?<timestamp>%{YEAR}-%{MONTHNUM}-%{MONTHDAY} %{TIME})  %{LOGLEVEL:level} %{NUMBER:pid} --- .+? :\s+(?<logmessage>.*)"
             ]
  }

  #Parsing out timestamps which are in timestamp field thanks to previous grok section
  date {
    match => [ "timestamp" , "yyyy-MM-dd HH:mm:ss.SSS" ]
  }
}
output {
  stdout{
   codec => rubydebug
  }
```
**2，写入log到文件app.log**

```
echo "2016-06-29 17:14:09.477  INFO 6244 --- [main] faultConfiguringBeanFactoryPostProcessor : No bean named 'errorChannel' has been explicitly defined. Therefore, a default PublishSubscribeChannel will be created." >> app.log

```
**3， 切换logstash查看输出**

```
{
       "message" => "2016-06-29 17:14:09.477  INFO 6244 --- [main] faultConfiguringBeanFactoryPostProcessor : No bean named 'errorChannel' has been explicitly defined. Therefore, a default PublishSubscribeChannel will be created.",
      "@version" => "1",
    "@timestamp" => "2016-06-29T16:14:09.477Z",
         "count" => 1,
        "fields" => nil,
        "source" => "/home/osboxes/app.log",
        "offset" => 987,
          "type" => "log",
    "input_type" => "log",
          "beat" => {
        "hostname" => "osboxes",
            "name" => "osboxes"
    },
          "host" => "osboxes",
          "tags" => [
        [0] "beats_input_codec_plain_applied"
    ],
     "timestamp" => "2016-06-29 17:14:09.477",
         "level" => "INFO",
           "pid" => "6244",
        "thread" => "main",
         "class" => "faultConfiguringBeanFactoryPostProcessor",
    "logmessage" => "No bean named 'errorChannel' has been explicitly defined. Therefore, a default PublishSubscribeChannel will be created."
}

```
至此，完成了初步的日志的解析，日志别解析至对应的fields中。 接下来将这些数据推送至Elasticsearch进行索引。

### 修改logstash配置，输出到elasticsearch
修改配置文件的output。 

```
output {
    elasticsearch {
    }
}
```
用这样的结构，Logstash使用http协议连接到Elasticsearch。上面的例子假设Logstash和Elasticsearch运行在同一个机器上。您可以使用主机配置`hosts => "es-machine:9092`指定远程Elasticsearch实例。
### 查看结果
一次启动elasticsearch，kibana，logstash，filebeat。 （filebeat已启动的话，无需重启）

### 安装Sense
进入/opt/kibana/ 运行：`$sudo ./bin/kibana plugin --install elastic/sense`
You should now be able to access Sense with a web browser on http://localhost:5601/app/sense



## spring boot 日志配置
**尽量采用统一的日志输出格式**
1, JPA 的sql输出

```
#spring.jpa.show-sql = true #不推荐这种方式
logging.level.org.hibernate.SQL=DEBUG
```

## 常见的部署方式
由于logstash比较消耗系统资源， 采用filebeat 来采集数据， 然后推送到logstash。 简单的case可以将logstash elasticsearch  kibana 放在一个虚拟机。 filebeat可以分别安装在各个对应的微服务上。 **注意：**当这些部署在不同的机器上的时候，需要打开对应的端口。 对应的配置也需要相对修改下。
**打开logstash的端口：**

```
$ sudo firewall-cmd --zone=public --add-port=5044/tcp --permanent
$ sudo firewall-cmd --reload
```
**filebeat的配置修改**

```
logstash:
    # The Logstash hosts
    hosts: ["192.168.1.186:5044"]
```
**修改hostName**
如果微服务部署在不同的虚拟机中， 可以通过修改hostname，然后在ES的index中通过hostname 来区分日志的来源

```
$ hostnamectl status
# hostnamectl set-hostname Your-New-Host-Name-Here
```

## 关于日志采集的策略
（网上未提及此topic）
配置logstash是件麻烦事情。 一下两种策略互相冲突
**1， 保证所有的log都index到ES**
这中策略方便用户查找问题， 因为所有的log都可以搜索到
**2， 严格过滤， 只提取我们需要的log信息**
这种很方便做统计， 但是其他很多log会被过滤掉， 用来找问题不方便。

## 服务器时间设置
最好保证日志源的服务器时间和ELK的数据库服务器时间一直
```
# ln -s /usr/share/zoneinfo/Asia/Shanghai /etc/localtime
```
