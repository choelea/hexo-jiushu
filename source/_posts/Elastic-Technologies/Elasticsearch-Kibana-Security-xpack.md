---
title: Kibana 5.x 加强安全 - x-pack篇
description: 使用插件x-pack来加强Kibana 5.x 的访问控制
---
 此文之前，假定读者已经一次完成了Kibana和elasticsearch的安装。参考官方文档，安装后默认配置已经可以连通kibana和es。 
 
 - 系统： centos7
 - 内容： 增加authentication & enable ssl
 
 elastic 技术栈 的另外一个重要的角色是x-pack. 

![elastic-charm](http://tech.jiu-shu.com/Elastic-Technologies/elastic-charm.png)

### ES安装xpack插件
参考[安装xpack](https://www.elastic.co/guide/en/x-pack/current/installing-xpack.html)
Run bin/elasticsearch-plugin install from ES_HOME on each node in your cluster:
```
bin/elasticsearch-plugin install x-pack
```

### Kibana 安装xpack 插件
参考[安装xpack](https://www.elastic.co/guide/en/x-pack/current/installing-xpack.html)

Install X-Pack into Kibana by running bin/kibana-plugin in your Kibana installation directory.
```
bin/kibana-plugin install x-pack
```
### 依次启动elasticsearch 和kibana


### 修改用户elastic 和 kibana的密码
[X-Pack 文档：修改密码](https://www.elastic.co/guide/en/x-pack/current/security-getting-started.html)
> X-Pack security provides a built-in elastic superuser you can use to start setting things up. The default password for the elastic user is changeme.

```
curl -XPUT -u elastic 'localhost:9200/_xpack/security/user/elastic/_password' -d '{
  "password" : "elasticpassword"
}'
```
```
curl -XPUT -u elastic 'localhost:9200/_xpack/security/user/kibana/_password' -d '{
  "password" : "kibanapassword"
}'
```
> CURL授权
在访问需要授权的页面时，可通过-u选项提供用户名和密码进行授权。 通常的做法是在命令行只输入用户名，之后会提示输入密码，这样可以保证在查看历史记录时不会将密码泄露

### Enable Kibana SSL
[Using Kibana in a Production Environment](https://www.elastic.co/guide/en/kibana/current/production.html)
配置上证书的路径即可：
```
# SSL for outgoing requests from the Kibana Server (PEM formatted)
server.ssl.key: /path/to/your/server.key
server.ssl.cert: /path/to/your/server.crt
```
修改了超级用户的密码，enable ssl后，就可以放心的去使用kibana的**Dev Tools** 或者chrome插件（sense）进行大部分API 的操作。 （在此之前需要ssh到服务器通过curl来操作以保证安全）
### 创建用户logstash_writer
[官方参考](https://www.elastic.co/guide/en/x-pack/5.1/logstash.html)
上面步骤完成后会发现logstash推送给es报错了。因为现在ES需要用户名和密码了。 这里我们需要创建一个用户拥有write, delete, and create_index的权限。

```
[2016-12-23T20:42:19,350][WARN ][logstash.outputs.elasticsearch] Attempted to resurrect connection to dead ES instance, but got an error. {:url=>#<URI::HTTP:0x17b5a1bd URL:http://localhost:9200>, :error_type=>LogStash::Outputs::ElasticSearch::HttpClient::Pool::BadResponseCodeError, :error=>"Got response code '401' contact Elasticsearch at URL 'http://localhost:9200/'"}
[2016-12-23T20:42:20,132][WARN ][logstash.shutdownwatcher ] {}

```
- 先创建一个role：logstash_writer

```
POST _xpack/security/role/logstash_writer
{
  "cluster": ["manage_index_templates", "monitor"],
  "indices": [
    {
      "names": [ "logstash-*","business-index-*"], 
      "privileges": ["write","delete","create_index"]
    }
  ]
}
```

 - 再创建一个用户：logstash_internal拥有Role：logstash_writer

```
POST /_xpack/security/user/logstash_internal
{
  "password" : "changeme",
  "roles" : [ "logstash_writer"],
  "full_name" : "Internal Logstash User"
}
```
> 上面的操作也可以通过Kibana的Management UI来操作

- 配置logstash.conf

```
output {
  elasticsearch {
    ...
    user => logstash_internal
    password => changeme
  }
```

> logstash, elasticsearch, kibana 如果在同一网络，而暴露出去的只有kibana的话，logstash和elasticsearch 之前是无需授权的。可以参考[Enabling Anonymous Access](https://www.elastic.co/guide/en/x-pack/current/anonymous-access.html) 另外，logstash和elasticsearch之间如果需要授权，会不会有性能的影响？

### 给Kibana用户加上index的读的权限
Kibana安装xpack后默认就需要登录了。也可以用超级用户elastic登录
登录后打开DevTools进行ES API的操作。


修改后停掉kibana服务。修改kibana的配置：
> Once you change the password, you need to specify it with the elasticsearch.password property in kibana.yml:

```
elasticsearch.password: "s0m3th1ngs3cr3t"
```

 
### 坑 （Tricky Part）

 1. /etc/logstash/conf.d 下不要有多余的文件。比如logstash.conf.bak， 似乎logstash会读这个文件夹下的不止logstash.conf这个文件配置。logstash.conf.bak 会导致死循环一样的重启。[elastic community](https://discuss.elastic.co/t/logstash-endless-loop-with-starting-and-stopping/69913)