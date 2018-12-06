---
title: Elasticsearch 自定义Mapping
description: Elasticsearch 自定义Mapping
---
## Mapping 定义
前面有一个篇简单的关于mapping的博客，当时是基于2.4 版本。 elastic技术栈在最近很活跃，目前版本已经更新至5.x。5.x有了比较大的变化。2.4 版本的定义在5.x上大部分已经失去了意义。（比如：[5.x已经不再支持string 类型](https://www.elastic.co/guide/en/elasticsearch/reference/5.0/string.html)）
**这里截取一点官网对应的**[定义](https://www.elastic.co/guide/en/elasticsearch/reference/5.0/mapping.html)：
 > elasticsearch 通过定义的映射mapping来决定文档及其字段改如何被存储和索引。比如：字段是否可以支持全文搜索; 字段是否包含日期，地理位置; 日期的格式; 自定义自动映射的规则。

基于5.x，[前面博客](http://blog.csdn.net/choelea/article/details/53320140) 提到的user，uri等字段就可以使用[keyword type](https://www.elastic.co/guide/en/elasticsearch/reference/5.0/keyword.html)。

```
PUT /business-index-*/_mapping/business
{
 "properties" : {
    "uri" : {"type": "keyword"},
    "user" : {"type": "keyword"},
    "keyword" : {"type": "keyword"},
    "responseStatus" : { "type" : "integer" },
    "responseTime" : { "type" : "long" }
 }
}
```

elastic的文档维护的算是比较好的，基本英语OK的都是直接去参考官方文档。  mapping的更新可以参考 [elastic 官网](https://www.elastic.co/guide/en/elasticsearch/reference/5.0/indices-put-mapping.html)	 

