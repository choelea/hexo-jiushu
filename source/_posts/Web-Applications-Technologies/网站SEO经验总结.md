---
title: 网站SEO优化最佳实践
description: 网站SEO优化最佳实践
...

# 网站 SEO 实践及注意事项
SEO着重于提高页面的收录率及网站在搜索引擎中的排名。
## 代码规范
1.	确保网页文件返回的文件头正确。 如：如果是404页面就一定是返回404文件头，如果返回错误，就会出问题的。也不要返回多个文件头。
2.	代码中，除了<!DOCTYPE声明外，请一律用小写；代码中不要有多余的换行和空格，这样凭空增加了页面的大小。
3.	页面要尽量符合W3C标准。 如果不能做到的情况下，html标签一定要关闭，否则搜索引擎会直接略过它不能确定的内容；属性值必须带上英文双引号。
4.	禁止使用隐藏链接。会被搜索引擎认为隐藏链接的做法，就是 <a>标签之间是空的，没有图片或者文字。比如：<a href="****"></a>
5.	请尽量把页面中的CSS代码和 javascript代码外调。不能外调的，请尽量把代码往下挪。
6.	一个网页中所有的链接都要尽量用绝对路径，除非影响性能或者一定需要用相对路径不可。
7.	页面代码中，写在javascript中的链接，搜索引擎是可以读的，所以有什么不想搜索引擎收录的链接，万一要写在JS代码中的话，请把JS外调。
8.	DIV的嵌套是不可避免的，但是在有嵌套的时候，请选择嵌套层级最少的那个方法。

##  URL 规范
1.	URL 全部小写；
2.	URL中最好有关键词的存在，并且关键词之间用中划线连接；
3.	URL 尽可能静态化，参数最多不能超过3个(比如： products?category=shoes 可以静态化为/shoes/product, 对应的pattern为：/{category}/products)
4.	使用 rel="canonical" 链接元素指明首选网址。

## 页面H标签的使用
H标签在页面代码当中起着引导性作用，对于搜索引擎来说，标签的存在意义是为了让搜索引擎定位并识别到这一部分的文字或是重点主题的内容。对网页的重要内容而言，它起着“强调作用”。 
 
 在实际的运用当中，由于H1-H6标签文字是由大至小，其重要与权重的高低程度是按排号依次递减的，因此我们都会对这些标签的使用进行分配：H1（内容主题）、H2（段落标题）、H3（小段落标题）以此类推至H6标签。 

参考：链接：https://www.chinaz.com/web/2015/0730/428499.shtml 

## SiteMap
1.	网站必须有sitemap
2.	根据语言的二级域名要分别创建各自的sitemap
3.	sitemap中的<urlset> <url> <loc> 必填

## 多语言
1.	<html lang="en"> 标记当前页面语言
2.	使用标签表明当前页面内容的语言 <meta http-equiv="content-language" content="pt" />
3.	使用 hreflang 设置语言和区域网址， 例如：
```
<link rel="alternate" hreflang="en" href="https://www.okchem.com/" />
<link rel="alternate" hreflang="es" href="https://es.okchem.com/" />
<link rel="alternate" hreflang="pt" href="https://pt.okchem.com/" />
	```

## 图片相关 ##
1.	对于网页中图片的高或宽大于200像素的图片，一定要在代码中看到图片地址而且写上这张图片的高宽大小，以及alt文本和 title文本。 `<img height="290" src=" " width="295" alt=" " title=" " />` alt和title可以一样，但是不能堆砌关键词。
2.	当链接对象是图片的时候，图片要有alt和title属性 `<a href="****" title=" "> <img src=" " alt=" " /> </a>`

## 网站响应时间
网站相应时间无论从用户体验或者SEO的角度来讲都很重要。影响响应时间有很多因素： 1. 利用浏览器缓存;
2. 图片大小；一般情况所有图片必须经过压缩处理；一般来讲banner不能超过100k，缩略图不能超过15K。 优化图片可以通过压缩大图，合并小图减少网络请求 3. JS和CSS； 过多和过大的JS和CSS资源也同样会影响网页速度。合并压缩JS和CSS；适当异步加载JS都可以提高相应速度。（异步加载的JS不参与页面的渲染） 4. CDN 缓存静态资源； 5. 独立静态资源的服务，提高浏览器并发

Etag  和 Cache Controller 都需要
etag 是在服务器端做对比
Cache Controller 可以让图片请求直接不去请求服务器
## 网站管理事项 ##
1.	网页上已存在的链接，如果需要撤消或更改，需要让SEO人员知道并确认；最好的情况是不要随便删除。变更URL的时候使用301重定向跳转，不要使用302。
2.	链接的文本要和被链接的页面内容保持一致。
3.	可用性及响应速度； 不能宕机或者经常出现404页面
4.	擅用Robots.txt 文件。网站后台不需要展现给搜索引擎的目录，请用Robots.txt文件屏蔽。 应该形成习惯，把不需要让搜索引擎知道的目录，在网站的根目录下的robots.txt文件中屏蔽它。
5.	要用任何阻止大量爬虫访问的系统。 比如防采集系统不要用。虽然爬虫确实占了很大一部分的带宽，但是是有价值的。

## 网站是否存在死链
使用xenu 这个工具可以扫面网站是否存在死链接

## 注意事项
1.	慎用代码copy。 网站上的某些代码，对于网站功能方面是不会有影响的。但是可能是对搜索引擎爬虫有影响的。所以以下的代码copy是要注意的：比如：`<title></title> <meta> <a href= rel=”nofollow”>`

## TDK
TDK 规则（待补充）
> 不同的页面要避免重复的tdk，如果重复会导致google不认你设置的tdk。 [Why Won't Google Use My META Description](https://moz.com/blog/why-wont-google-use-my-meta-description)

## Google SEO 相关链接
* Google SEO的帮助文档
* google可以识别的元标记 https://support.google.com/webmasters/answer/79812, （google 并不认可keyword的meta信息 https://webmasters.googleblog.com/2016/11/saying-goodbye-to-content-keywords.html?hl=zh-Hans&visit_id=1-636691380412463003-1651263000&rd=1）



# 其他参考

## 移动前端不得不了解的HTML5 head 头标签 

https://www.cnblogs.com/happiness-mumu/p/6054852.html

# 分析工具
https://gtmetrix.com/reports/www.okchem.com/3HaT9qRN