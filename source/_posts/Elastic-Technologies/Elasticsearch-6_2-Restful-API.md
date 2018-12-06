---
title:  Elastcisearch 6.2 Restful API 
description: Elastcisearch 常用的 Restful API
...

# Elastcisearch
详细的API请参考官方网站： https://www.elastic.co/guide/en/elasticsearch/reference/current/index.html 这里只列举常用的方式。

## 索引API
官方链接： [https://www.elastic.co/guide/en/elasticsearch/reference/6.2/indices.html](https://www.elastic.co/guide/en/elasticsearch/reference/6.2/indices.html)
### 创建索引
#### 快速创建

```
PUT /news
```
创建名为test的索引，没有创建任何对应的Type,以及Mapping
```
{
  "acknowledged": true,
  "shards_acknowledged": true,
  "index": "news"
}
```

### 查看索引
```
GET /news
```
```
{
  "news": {
    "aliases": {},
    "mappings": {},
    "settings": {
      "index": {
        "creation_date": "1535677066065",
        "number_of_shards": "5",
        "number_of_replicas": "1",
        "uuid": "-fZX17QdQjWE_AK79pO8lQ",
        "version": {
          "created": "6020499"
        },
        "provided_name": "news"
      }
    }
  }
}
```

### 删除索引
```
curl -XDELETE "http://192.168.1.99:9200/news" // 删除索引
```
```
{
   "ok": true,
   "acknowledged": true
}
```
#### 设置类型并定义Mapping (推荐)

```
PUT /news/_mapping/_doc
{
   "properties":{
     "title":{
       "type":"text"
     },
     "content":{
       "type":"text"
     },
     "postDate":{
       "type":"date"
     },
     "categories":{
       "type":"keyword"
     },
     "tags":{
       "type":"keyword"
     }
   }
}
```
1. title和content是用于全文检索的，同时需要分词的
2. categories tags无需分词，这里的categories和tags都会存放多个值的数组。关于数组类型参考[Array DataType](https://www.elastic.co/guide/en/elasticsearch/reference/current/array.html)

> elasticsearch 支持 [Dynamical Mapping](https://www.elastic.co/guide/en/elasticsearch/reference/current/dynamic-mapping.html), 大多数情况下，这都不是一个推荐方式。


