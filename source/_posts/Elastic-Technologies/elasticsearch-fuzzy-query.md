---
title:  Elasticsearch 模糊匹配
description: fuzzy 查询是一个词项级别的查询，所以它不做任何分析。它通过某个词项以及指定的 fuzziness 查找到词典中所有的词项。 fuzziness 默认设置为 AUTO 。
...

这个是一个模板, 请务必将showOnHome 修改为true


https://www.elastic.co/guide/cn/elasticsearch/guide/cn/fuzzy-scoring.html
以下来自
```
模糊性评分编辑
用户喜欢模糊查询。他们认为这种查询会魔法般的找到正确拼写组合。 很遗憾，实际效果平平。

假设我们有1000个文档包含 ``Schwarzenegger`` ，只是一个文档的出现拼写错误 ``Schwarzeneger`` 。 根据 term frequency/inverse document frequency 理论，这个拼写错误文档比拼写正确的相关度更高，因为错误拼写出现在更少的文档中！

换句话说，如果我们对待模糊匹配 类似其他匹配方法，我们将偏爱错误的拼写超过了正确的拼写，这会让用户抓狂。
```