---
title: Logstash Filter 配置
description: Logstash Filter 配置
---
笔者这里仅仅列出配置文件，在研究之后最红并没有采用在logstash的接下日志为json的做法。而是将json的输出放在了各个服务/应用中处理， spring boot的app可以参考：[logstash-logback-encoder](https://github.com/logstash/logstash-logback-encoder)
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
    match => [ 
				#	Record transaction
				"message","(?<timestamp>%{YEAR}-%{MONTHNUM}-%{MONTHDAY} %{TIME})  %{LOGLEVEL:level} %{NUMBER:pid} --- \[\s*(?<thread>[^\]]+)\] (?<class>[A-Za-z0-9.#_]+)\s*: \[\s*(?<transactionInfo>[^\]]+)\]",
				"message", "(?<timestamp>%{YEAR}-%{MONTHNUM}-%{MONTHDAY} %{TIME})  %{LOGLEVEL:level} %{NUMBER:pid} --- \[\s*(?<thread>[^\]]+)\] (?<class>[A-Za-z0-9.#_]+)\s*:\s+(?<logmessage>.*)",
				"message", "(?<timestamp>%{YEAR}-%{MONTHNUM}-%{MONTHDAY} %{TIME})  %{LOGLEVEL:level} %{NUMBER:pid} --- .+? :\s+(?<logmessage>.*)"
             ]
  }

  #Parsing out timestamps which are in timestamp field thanks to previous grok section
  date {
    match => [ "timestamp" , "yyyy-MM-dd HH:mm:ss.SSS" ]
  }
}
output {
 elasticsearch{} 
 stdout{
   codec => rubydebug
  }
}

```

这里grok配置了三册过滤， 第一层用作统计，message的格式如下：

```
2016-07-15 20:30:30.884  INFO 14624 --- [nio-8081-exec-3] c.l.a.w.controller.OfbizProxyController  : [{"transactionCode":"ofbizProxy","transactionDuration":246}]
```
使用[Grok Debugger](http://grokdebug.herokuapp.com/) 解析后如下

```	Json
{
  "timestamp": [
    [
      "2016-07-15 20:30:30.884"
    ]
  ],
  "YEAR": [
    [
      "2016"
    ]
  ],
  "MONTHNUM": [
    [
      "07"
    ]
  ],
  "MONTHDAY": [
    [
      "15"
    ]
  ],
  "TIME": [
    [
      "20:30:30.884"
    ]
  ],
  "HOUR": [
    [
      "20"
    ]
  ],
  "MINUTE": [
    [
      "30"
    ]
  ],
  "SECOND": [
    [
      "30.884"
    ]
  ],
  "level": [
    [
      "INFO"
    ]
  ],
  "pid": [
    [
      "14624"
    ]
  ],
  "BASE10NUM": [
    [
      "14624"
    ]
  ],
  "thread": [
    [
      "nio-8081-exec-3"
    ]
  ],
  "class": [
    [
      "c.l.a.w.controller.OfbizProxyController"
    ]
  ],
  "transactionInfo": [
    [
      "{"transactionCode":"ofbizProxy","transactionDuration":246}"
    ]
  ]
}
```

第二层针对普通的log

```
2016-07-15 20:30:07.768  INFO 14624 --- [nio-8081-exec-1] c.l.a.web.controller.LoginController     : Login username:vincent.chen@okchem.com IP is:0:0:0:0:0:0:0:1
```
解析后的json如下：

```
{
  "timestamp": [
    [
      "2016-07-15 20:30:07.768"
    ]
  ],
  "YEAR": [
    [
      "2016"
    ]
  ],
  "MONTHNUM": [
    [
      "07"
    ]
  ],
  "MONTHDAY": [
    [
      "15"
    ]
  ],
  "TIME": [
    [
      "20:30:07.768"
    ]
  ],
  "HOUR": [
    [
      "20"
    ]
  ],
  "MINUTE": [
    [
      "30"
    ]
  ],
  "SECOND": [
    [
      "07.768"
    ]
  ],
  "level": [
    [
      "INFO"
    ]
  ],
  "pid": [
    [
      "14624"
    ]
  ],
  "BASE10NUM": [
    [
      "14624"
    ]
  ],
  "thread": [
    [
      "nio-8081-exec-1"
    ]
  ],
  "class": [
    [
      "c.l.a.web.controller.LoginController"
    ]
  ],
  "logmessage": [
    [
      "Login username:vincent.chen@okchem.com IP is:0:0:0:0:0:0:0:1"
    ]
  ]
}
```
第三层针对遗漏的无法匹配到的log再次解析， 这里暂时没有示例