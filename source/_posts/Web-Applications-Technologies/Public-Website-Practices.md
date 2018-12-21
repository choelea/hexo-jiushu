---
title: 对公网站研发的实践经验
description: 对公网站研发的实践经验及需要注意事项
...

有过很多的面向员工系统经验的工程师，在开发一个面向客户的web网站的时候会面临和需要考虑的问题更多，也更复杂。下面列举在开发对公网站时候需要考虑的一些问题和实践。

### 是否有SEO 需求
当有SEO需求的时候，在选择前端框架的时候，需要注意对SEO的支持。 各种流行的框架： React和Angularjs 等SPA的页面框架大部分在SEO方面都多少有些问题
SEO 优化：

 1. URL 静态化，可读性 (比如： products?category=shoes 可以静态化为/shoes/product, 对应的pattern为：/{category}/products)
 2. 页面布局，图片的URL，图片的alt信息 等都需要考虑
 3. 网站内的内链
 3. 异步使用的恰当，简单的链接更有助于网站收录
> SEO 的详细介绍请参考：[网站SEO优化最佳实践](http://tech.jiu-shu.com/Web-Applications-Technologies/%E7%BD%91%E7%AB%99SEO%E7%BB%8F%E9%AA%8C%E6%80%BB%E7%BB%93)

### 网页速度
网页相应速度涉及用户体验，同时也会影响到SEO的排名， 可以通过[Google Page Insights](https://developers.google.com/speed/pagespeed/insights/) 来检查网页速度相关问题。 一般优化网页速度的包括：

1.	优化图片。 优化图片一般都是性价比高的提速方案。 优化图片分为：
	-	优化格式，压缩大小。 （之前的做了一次产品的旧图片的压缩，但是还有其他图片需要进一步排查压缩，比如banner）
	- 合并小图片，减少网络请求。将一个个小图片合并成一个大图片，用css 控制显示范围。
	- 懒加载, 针对页面上图片比较多的情况， 比如转灯（bootstrap的carousel），默认情况是所有的图片请求都一直并发；当图片过多的时候对页面的相应影响很大，这个时候要考虑图片的懒加载机制。懒加载参考：[页面懒加载机制](/Web-Applications-Technologies/image-lazy-loading)
2.	合并压缩JS 和 CSS 。 （减小文件大小，减少网络请求）
3.	CDN 的引入。 暂时不考虑付费的CND，但是针对jquery及bootstrap等知名的第三方资源，可以使用免费的CDN. (具体依赖预生产的测试来做最终决定)
4.	代码优化。 代码优化比较难在短时间完成。但是可以逐步挑选重要的页面来优化：
a)	按需加载， js 引入位置，延迟/异步加载阻塞的js文件。
b)	优化cookie，减少cookie的体积
c)	Js css规范化
5.	Nginx优化。 在查看webpagetest的结果的时候，发现https的握手时间也占了不少比例，针对nginx的ssl的优化也是必须的。（大数据新闻的图片慢有可能和这个有关，具体需要通过测试来验证） （made-in-china 网站仍旧采用的是http，在速度上有很大优势）
6.	将静态资源的请求放入另外的domain来提高浏览器请求的并发 （效仿made-in-china 的做法）
7.	页面静态化处理。 前面讲到的都和服务端代码无关，页面静态化处理主要是在服务端提前将页面生成静态的html文件，提高并发。 目前服务器相应时间并不是主要拖慢网页的因素，因此这个优先级不高。
 
**综合来讲，主要思想是： 减少请求次数，减少文件大小，延迟请求，异步加载，CDN缓存，最小化https的开销。 最后在完成这些之后，更深一步去规范页面内容，JS CSS 最佳实践。**

### 防止攻击
#### Bot Attack 
如果网站里面有开放的POST接口，通过过一段时间就会遭遇Bot 攻击。
通常，添加验证码会是办法之一。 参考讨论[Preventing bot form submission](https://stackoverflow.com/questions/15319942/preventing-bot-form-submission) 采用google的[recaptcha](https://www.google.com/recaptcha/intro/) 也会是一个不错的方案。 
[开发者文档](https://developers.google.com/recaptcha/intro)



###  下载相关
文件下载最好不要放在网站内， 可以放在云服务器上， 比如aws的s3， 或者其他云对象服务器上。
注意download的response 相关的header设置等。 
比如： apk的下载， 有些浏览器无法智能的根据文件名后缀.apk来判断是安卓的安装程序。需要显示的设置： content-type： application/vnd.android.package-archive

### 需要更多的手段来验证网站的可用性
面向员工的系统，如果系统不可用，或者有任何问题发生，员工都会立马反应。（很可能他现在要提交个工单，提交不了，着急啊。。。） 然后面向光大客户的就不是了。 比如说， 淘宝现在用不了了，系统报错，你会想淘宝反应吗？ 我想你会去京东买了算了。。。



 
