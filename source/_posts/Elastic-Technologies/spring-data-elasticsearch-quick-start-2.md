---
title:  Spring Data Elasticsearch 快速上手全文检索 - 进阶
description: 通过Spring Data Elasticsearch 实现全文检索并高亮关键词。
...

继上一篇 [Spring Data Elasticsearch 快速上手全文检索](http://tech.jiu-shu.com/Elastic-Technologies/spring-data-elasticsearch-quick-start)之后，进一步深入以下内容：
* 高亮显示关键词
* 指定Analyzer更合理的检索

> 最新的master的代码升级Spring Boot到1.5.13.RELEASE， 对应的spring-data-elasticsearch 自动升级至2.1.12.RELEASE， 在此版本基础上，DefaultResultMapper 已经支持了聚合。无需为聚合儿自定义ResultMapper。

## 代码
```
git clone https://github.com/choelea/spring-data-elasticsearch-quick-start
```

https://github.com/elastic/elasticsearch/issues/11713

## 高亮关键词
对name和description中的关键字进行高亮显示，直接参考代码：
```
SearchQuery searchQuery = new NativeSearchQueryBuilder().withQuery(queryBuilder)
				.withPageable(pageable)
				.withHighlightFields( new HighlightBuilder.Field(ProductDoc._name).forceSource(true), new HighlightBuilder.Field(ProductDoc._description).forceSource(true))
				.addAggregation(termBuilder).build();
```
默认情况下返回高亮字段不在_source内，当转成成我们的ProductDoc的时候对应的name和description是不会有变化的， 这个时候还是需要定制ResultMapper， 因此这里定制了一个ExtResultMapper。 将高亮字段覆盖到ProductDoc 中对应的字段去。