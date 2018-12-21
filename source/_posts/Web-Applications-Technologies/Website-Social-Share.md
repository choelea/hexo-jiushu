---
title: 社交分享在网站中的实现
description: 社交分享在网站中的实现的简单说明
---
公司的web项目中，经常会有社交分享，这里简单说明下，方便后面开发人员理解。

## 分享的原理
社交分享(linkedin, facebook, twitter)的本质就是请求其一个公开的页面。通过参数告诉社交网站你需要分享的网页的网址，然后社交网站的爬虫会去爬取这个网址。显然，你分享的网页必须是可以**匿名**从**公网**访问的；爬虫爬取和谷歌，百度的爬虫是一个道理，只是各自算法不一样。
常用的社交网站的分享链接如下：
 - Linkedin http://www.linkedin.com/shareArticle?url=https://www.okchem.com/instant-quote
 - Facebook http://www.facebook.com/sharer.php?u= https://www.okchem.com/instant-quote
 - Twitter http://twitter.com/share?url=https://www.okchem.com/instant-quote
更多参数可以参考相关文档，但是基本上这个参数就行了

## 分享内容定制
一般情况下，类似谷歌和百度的爬虫，会爬取title description （和SEO中的一样）还有图片等作为缩略展示。 因为各自算法的原因，并不是100%能保证你的title和description就会被展示出来；也不能保证你想要出现的图片就以一定会出现在缩略中。
如果想**告知**爬虫你的网页的title description和图片等，可以通过Open Graph Meta Tags 来**指引**爬虫。 具体可以参考[http://ogp.me/](http://ogp.me)， 详情请去百度和google。
示例：

```
<meta property="og:title" content="[Hot Item] Ce Approved Superfine Synthetic Graphite Spheroidization Grinding Mill"/>
<meta property="og:type" content="product"/>
<meta property="og:url" content="http://leap-tech.en.made-in-china.com/product/gBvQXLzMCKcJ/China-Ce-Approved-Superfine-Synthetic-Graphite-Spheroidization-Grinding-Mill.html"/>
<meta property="og:image" content="http://image.made-in-china.com/2f0j00sJdQPUDMngkI/Ce-Approved-Superfine-Synthetic-Graphite-Spheroidization-Grinding-Mill.jpg"/>
<meta property="og:site_name" content="Made-in-China.com"/>
<meta property="og:description" content="This site would be your best choice when sourcing from China. Their buyer service is professional and it offers third party transaction service to protect your money"/>
```
摘自网站http://leap-tech.en.made-in-china.com/product/gBvQXLzMCKcJ/China-Ce-Approved-Superfine-Synthetic-Graphite-Spheroidization-Grinding-Mill.html 通过og标签分别告知爬虫这个页面的title, url, image, description。 当页面有很多图片，可以通过og:image来告知社交网站采用哪张图片做缩略显示。
> 前面的描述中，都用了告知，引导。 就像谷歌爬虫一样，社交网站的爬虫也只是**尊重**你的页面的设置，但是他们更依赖自己的算法。所以并不会100% follow。

## 注意事项
在测试的时候很容易碰见，分享的时候没有看见缩略图，内容没更新等等问题。很可能和各自的缓存有关，也可能是你的页面的内容不对。
Facebook有调试工具：https://developers.facebook.com/tools/debug/sharing
https://developers.facebook.com/docs/sharing/best-practices
> 分享facebook的时候有可能预览的图片不能及显示，方案可以参考: https://developers.facebook.com/docs/sharing/best-practices#precaching

Linked in 的分享说明参考：https://developer.linkedin.com/docs/share-on-linkedin# 注意最后提到的缓存。
![Linkedin Share](http://tech.jiu-shu.com/Web-Applications-Technologies/linkedin-share-cache.png)

Twitter的工具参考：https://cards-dev.twitter.com/validator

**要想分享在Twitter上正常显示，下面的标签必须全部都加上。**
```
<meta property="og:type" content="website">
<meta property="og:image" content="your page image url"/>
<meta property="og:title" content="Title of the page"/>
<meta property="og:description" content="Description of your page"/>
```

