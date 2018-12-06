---
title: Metricbeat 的使用
description: 通过Metricbeat 来统计并展示系统的信息 cpu， 内存等
---
### 目标
统计并展示系统的信息 cpu， 内存等 (当然metricbeat能收集的信息种类还很多)
### 前提
 1. 版本： 5.x
 2. 已经安装了ELK (elasticsearch, logstash (可选）, kibana)
 3. 安装了x-pack  （配置了对应的security）（可选） 参考 [Kibana 5.x 加强安全](http://blog.csdn.net/choelea/article/details/53841218)

### 安装配置
安装，配置参考  [官方网站](https://www.elastic.co/guide/en/beats/metricbeat/current/metricbeat-installation.html)
使用OOTB配置即可，一般只需要修改ES的端口和地址。 如果加强了security，也需要更改 metricbeat.yml。 这里已经加强了安全，配置了用户，故需要更改metricbeat.yml添加elasticsearch的相关访问用户。
（创建角色和用户可以参考 [Kibana 5.x 加强安全](http://blog.csdn.net/choelea/article/details/53841218) ，这里角色需要用操作索引metricbeat-*）
> elasticsearch 默认绑定了localhost的访问，需要取消这种绑定。 设置`network.host: 0.0.0.0` 0.0.0.0 表示任意地址，如果设置成了IP地址，那么同台机器的kibana和logstash的需要做对应的修改。（比如：192.168.1.50， logstash和kibana需要把链接elasticsearch的hosts 从localhost改成：192.168.1.50）

### 加载kibana的示例 index template 和 dashboards
> 因为metricbeat 可能装在多个机器，index template 和dashboard 只需要导入一次即可。默认会自动加载index template到elasticsearch。

```
./scripts/import_dashboards -es http://localhost:9200 -user elastic -pass changeme
```

### kibana中查看对应的结果

登录kibana打开对应的dashboard 既可以看到统计报告了

![Kibana 中 展示系统运行状态](http://tech.jiu-shu.com/Elastic-Technologies/kibana-statics.png)

