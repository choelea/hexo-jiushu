---
title: Mongodb 和 Mysql 的性能测试
description: 基于Windows 7上来测试Mongodb和Mysql在一对多的场景下的性能
...
尝试测试Mongodb 和 Mysql的性能，测试/数据导入代码：[github: mongo-vs-mysql](https://github.com/choelea/mongo-vs-mysql)
> 性能比较很复杂，不能简单就说谁的性能高，谁的低。要基于场景，基于并发请求数量来谈，同时也要知道如何调优，本文只是初探，在没有任何调优的基础上，在本地windows 7上进行测试。
# 版本及环境

 - 操作系统：  windows 7 
 - 硬件环境： （只做对比，mongodb和mysql都装在同一台机器上） 
 - mongodb：  3.2.5
 - mysql：    5.7

# Data-demo
Data-demo 是一个Spring Boot的项目， 通过Spring Boot的CommandLineRunner来批量动态插入1000,020 条数据。

数据结构采用常用的产品和类目的多对多的设计。 
![mysql-product-category](http://tech.jiu-shu.com/Database-Technologies/mysql-product-category.png)

**Category 数据如下：**

id | code | name  
---|-----|----
'1'| 'cate-1'| 'Category 1'
'2'| 'cate-2'| 'Category 2'
'3'| 'cate-3'| 'Category 3'
'4'| 'cate-4'| 'Category 4'

**Product 数据如下**

id | code | name  | price
-- | ---- | ----- | -----
'1'| 'p-0'| 'product 0'| '19'
'2'| 'p-1'| 'product 1'| '19'
'3'| 'p-2'| 'product 2'| '19'
 ...|  ... |  ...          |...
'1000000'| 'p-999999'| 'product 999999'| '19'
'1000001'| 'pp-0'| 'iphone'| '19'
'1000002'| 'pp-1'| 'iphone'| '19'
'1000003'| 'pp-2'| 'iphone'| '19'
'1000004'| 'pp-3'| 'iphone'| '19'
'1000005'| 'pp-4'| 'iphone'| '19'
'1000006'| 'pp-5'| 'iphone'| '19'
'1000007'| 'pp-6'| 'iphone'| '19'
'1000008'| 'pp-7'| 'iphone'| '19'
'1000009'| 'pp-8'| 'iphone'| '19'
'1000010'| 'pp-9'| 'iphone'| '19'
'1000011'| 'pp-10'| 'iphone'| '19'
'1000012'| 'pp-11'| 'iphone'| '19'
'1000013'| 'pp-12'| 'iphone'| '19'
'1000014'| 'pp-13'| 'iphone'| '19'
'1000015'| 'pp-14'| 'iphone'| '19'
'1000016'| 'pp-15'| 'iphone'| '19'
'1000017'| 'pp-16'| 'iphone'| '19'
'1000018'| 'pp-17'| 'iphone'| '19'
'1000019'| 'pp-18'| 'iphone'| '19'
'1000020'| 'pp-19'| 'iphone'| '19'	

最后的二十行是用来方便查询验证的。

**Product_Category**

中间mapping的表格


# Mongo 数据
采用了 Nodejs+express+mongoose 来导入mongo的数据. 项目express-mongoose-microservice-api-boilerplate中的config/test.env来配置mongo的数据库地址。`npm install` 然后运行命令`npm run  produceTestData` 可以初始化1000,020 条产品数据到mongodb。 数据类似mysql的产品数据：

产品 Product
```
{
	"_id": "59cb4952d44efa2eb45d4bf7",
	"code": "p-0",
	"name": "Product 0",
	"price": 19,
	"__v": 0,
	"categories": [
		"cate-1",
		"cate-2"
	]
}
```
有20条产品数据的categories中有cate-4

# 通过查询脚本直接测试：
通过Robomongo 连接Mongodb来测试，通过mysql的workbench来完成mysql的脚本查询。
## 场景一：查询单个类目下的产品
#### mongo 查询所有的cate-4 的产品
``` mongodb
db.getCollection('products').find({categories:'cate-4'}) // 初次查询1.321 秒 紧接着的两次查询大概0.791 秒
```
#### mysql 查询所有的cate-4 的产品

``` sql
SELECT * FROM  product p inner join product_category pc inner join category c on p.id=pc.product_id and pc.category_id=c.id where c.code ='cate-4'; -- 毫秒级，时间可以忽略不计, 产品和类目的code都是unique的索引，所以查询速度很快
SELECT * FROM  product p inner join product_category pc inner join category c on p.id=pc.product_id and pc.category_id=c.id where c.name ='Category 4'; -- 6.2秒，name不是索引，所以慢。（索引的用处毫无疑问，无需赘述）
```
## 场景二：查询多个类目下的产品
查询所有cate-4 加上 cate-5 的产品。（实际上cate-5并不存在，不过不影响测试）
#### mongod 
```
db.getCollection('products').find({categories:{$in:['cate-4','cate-5']}}) // 0.89 秒； 和查询cate-4的产品相差不多，都是全表扫描
```
#### mysql 
```
SELECT * FROM  product p inner join product_category pc inner join category c on p.id = pc.product_id and pc.category_id = c.id where c.code = 'cate-5' or c.code='cate-4'; -- 6.177 秒，
SELECT * FROM  product p inner join product_category pc inner join category c on p.id = pc.product_id and pc.category_id = c.id where c.code in('cate-5','cate-4'); -- 6.24 秒
```
通过上面的测试可以看出，mysql数据库在数据体量大的时候，用or或者in都有很严重的性能问题，可以考虑使用union来代替。一般电商平台的处理方式：如果是后台维护功能应该从业务上来避免这种场景，如果是前端面向用户的功能，需要引入搜索引擎 比如： elasticsearch

## 场景三：单表无索引
查询名称是iphone的产品
#### Mysql 
```
select * from product where name='iphone'; -- 0.546 秒，全表扫描
```
#### mongo

```
db.getCollection('products').find({name:'iphone'}) // 0.428 秒
```
全表扫描两者并无太大的差距。

## 场景四：单表索引
```
db.getCollection('products').find({code:'pp-1'})  
select * from product where code='pp-1';
```
一百万条数据，单表索引速度都是毫秒级，时间可以忽略不计。

## 场景五：单表索引字段使用In来查询
场景二我们提到了Mysql中关联表时使用in查询的效率问题。下面测试下单表的In查询效率, 通过测试我们可以发现单表针对索引的in的查询都是毫秒级的。
**mongodb** 历时0.004 sec. 索引被用上了
```mongodb
// 非ID的索引字段
db.getCollection('products').find({
    "code": {
        "$in": [
            "pp-0",
            "pp-1"
        ]
    }
})
// _id 主键的$in 查询
db.getCollection('products').find({
    "_id": {
        "$in": [
            ObjectId("59cb4952d44efa2eb45d4bf7"),
            ObjectId("59cb4952d44efa2eb45d4bf8")
        ]
    }
})
```
**Mysql ** 也是毫秒级，时间忽略不计

``` sql
select * from product where code in ('pp-0','pp-4'); // 0.0000 sec
```


