---
title: Elasticsearch 2.X 自定义字段的Mapping
description: Elasticsearch 2.X 自定义字段的Mapping
---
说到Mapping大家可能觉得有些不解，其实我大体上可以将Elasticsearch理解为一个RDBMS（关系型数据库，比如MySQL），那么index 就相当于数据库实例，type可以理解为表,这样mapping可以理解为表的结构和相关设置的信息（当然mapping有更大范围的意思）。

默认情况不需要显式的定义mapping， 当新的type或者field引入时，Elasticsearch会自动创建并且注册有合理的默认值的mapping(毫无性能压力)， 只有要覆盖默认值时才必须要提供mapping定义。
> 引用博客：http://blog.csdn.net/top_code/article/details/50767138
## 术语
term - individual word （拆分后的最小单词）
## Mapping 简介
[Elasticsearch Reference [2.4] » Mapping](https://www.elastic.co/guide/en/elasticsearch/reference/2.4/mapping.html)
Mapping是用来定义文档及包含字段的保存和索引的方式。
## Why
接触mapping是因为要收集除了log之外的业务信息。 业务log和系统log不同，很多的自定义字段，并将这些信息推送到单独的index。 最终目的是用过kibana的图形化的展示来统计和分析。当我们要统计比如：用户的访问排名（字段名：user：test@gmail.com）。 当没有设置任何mapping的时候，ES会采用动态mapping（Dynamic Mapping），针对String的字段默认的index方式是：analyzed。这种方式下，test@gmail.com 会被拆分成test和gmail.com(怎么拆分取决于用什么analyzer)。这样不便于统计，这里我们必须显示地去设置mapping。
 [Mapping parameters » index](https://www.elastic.co/guide/en/elasticsearch/reference/2.4/mapping-index.html) 
 > 通过kibana去选择analyzed的字段去做terms aggregation可以看到对应的warning信息
## 自定义mapping
可以通过API 去自定义mapping。 （这个最好在数据开始index之前，因为数据index的时候会动态设置mapping，再去修改会出现一些冲突）新增加的字段可以继续通过修改mapping来增加。  ES 支持一个index多个type，mapping可以针对单个type也可以针对index。
**示例：**

```
curl -XPUT http://localhost:9200/business-index-*/_mapping/biz -d '
{
 "properties" : {
    "uri" : {"type": "string","index" : "not_analyzed"},
    "user" : {"type": "string", "index" : "not_analyzed"},
	"keyword" : {"type": "string", "index" : "not_analyzed"},
    "responseStatus" : { "type" : "integer" },
    "responseTime" : { "type" : "long" }
 }
}';
```

## 自定义template
对于确定的index，通过mapping的方式就可以达到我们的目的。 比如： 商品的索引，这个index不会变，里面的数据document会增删改查，但是index始终在那里。 
但是对于类似log和数据分析的数据，这些数据会惊人的速度增加，如果放在一个index就不现实。 所以ELK就有了 "***time-based index pattern***" , 通过这种方式可以每天或者每月生成一个index文件。比如logstash的日志： `logstash-2016.08.20 ` 针对这种场景，就需要引入更高一层的配置: [Index Template](https://www.elastic.co/guide/en/elasticsearch/reference/2.4/indices-templates.html) 
设定自己的template的示例如下：

```
curl -XPUT http://localhost:9200/_template/business -d '
{
	"template": "business*",
	"settings": {
		"number_of_shards": 1
	},
	"mappings": {
		"_default_": {
			"properties": {
				"uri": {
					"type": "string",
					"index": "not_analyzed"
				},
				"user": {
					"type": "string",
					"index": "not_analyzed"
				},
				"keyword": {
					"type": "string",
					"index": "not_analyzed"
				},
				"responseStatus": {
					"type": "integer"
				},
				"responseTime": {
					"type": "long"
				}
			}
		}
	}
}
';
```
> The settings and mappings will be applied to any index name that matches the business* template

