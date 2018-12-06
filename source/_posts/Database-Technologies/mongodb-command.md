---
title:  MongoDB 命令 常用语句
description: Mongodb常用命令
...

mongodb 常用命令收集
## 导出JSON数据
```
mongoexport -h localhost:27017 -d guide-chem -c product --limit 10000 --skip 10000 --jsonArray -u okchem -p okchem -o /home/okchem/products.json
```
**参数说明**
* -h 指定host 和端口
* -d 指定db
* -c 指定collection
* --limit 导出多少条
* --skip 跳过多少条
* --jsonArray 保存为json数组
* -u 指定用户
* -p 指定密码
* -o  指定导出文件路径output

## 修改字段名称

```
db.集合名称.update({}, {$rename:{"旧键名称":"新键名称"}}, false, true)
```
**参数说明**

* 第一个false表示：可选，这个参数的意思是，如果不存在update的记录，true为插入新的记录，默认是false，不插入。 
* 第二个true表示：可选，mongodb 默认是false,只更新找到的第一条记录，如果这个参数为true,就把按条件查出来多条记录全部更新。
 

