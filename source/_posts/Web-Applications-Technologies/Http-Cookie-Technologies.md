---
title: Http Cookie 探索
description: 深入了解测试Http Cookie相关; Cookie的设置，domain path等的作用
---
Meta 信息不见了

参考：
http://blog.csdn.net/lijing198997/article/details/9378047
http://stackoverflow.com/questions/1062963/how-do-browser-cookie-domains-work
**Domain and Path**
作用：定义Cookie的生效作用域，只有当域名和路径同时满足的时候，浏览器才会将Cookie发送给Server。如果没有设置Domain和Path的话，他们会被默认为当前请求页面对应值。 
> Cookie with Domain=.example.com will be available for www.example.com
Cookie with Domain=.example.com will be available for example.com
Cookie with Domain=example.com will be converted to .example.com and thus will also be available for www.example.com
Cookie with Domain=example.com will not be available for anotherexample.com
跨域的请求是无法设置cookie的。 （）

**Example：**

### Tomcat Java Web App通过服务端来设置浏览器cookie
服务端在请求的返回中向客户端的浏览器添加cookie。 
> 示例的服务的context path 为/bee

```
@RequestMapping(value = "/cookietest", headers = "Accept=image/**", method = RequestMethod.GET)
public ResponseEntity<?> cookieTest(final HttpServletRequest request,HttpServletResponse response) {
	Cookie cookie1 = new Cookie("cookie-1",UUID.randomUUID().toString());
	cookie1.setDomain("diaoyouyun.com");
	cookie1.setPath("/");
	response.addCookie(cookie1);
				
	Cookie cookie2 = new Cookie("cookie2","Cookie2");
	cookie2.setDomain("www.diaoyouyun.com");
	cookie2.setPath("/bee/collect");
	response.addCookie(cookie2);
	
	Cookie cookie3 = new Cookie("cookie3","cookie3");
	cookie3.setDomain("www.example.com");
	response.addCookie(cookie3);
	
	Cookie cookie4 = new Cookie("cookie4","cookie4");
	response.addCookie(cookie4);
   	return ok();
}
```

### 测试一

请求 http://diaoyouyun.com/bee/cookietest 如下：
![cookietest](http://tech.jiu-shu.com/Web-Applications-Technologies/cookie-1.png)

那些cookie会被接受呢？访问http://diaoyouyun.com/bee 从下图可以看出
![Cookies_http://diaoyouyun.com](http://tech.jiu-shu.com/Web-Applications-Technologies/cookie-2.png)

访问http://www.diaoyouyun.com/bee/来查看有哪些cookie
![cookies](http://tech.jiu-shu.com/Web-Applications-Technologies/cookie-3.png)

**关于Java Tomcat 服务端Set-Cookie: 可以得出以下结论：**

 1. 不显示地设置domain，浏览器接受去当年请求的domain，但是前面不加点（.）。即：如果当前请求domain是example.com,那么这个cookie就不能被www.example.com 或者其他\***.example.com访问到.
 2. 显示设置domain，只有设置正确的情况，cookie才会被浏览器接受

### 测试二

> 测试前清空相关站点的cookie

通过请求：http://www.diaoyouyun.com/bee/cookietest 来设置cookie
![Request](http://tech.jiu-shu.com/Web-Applications-Technologies/cookie-4.png)
访问http://www.diaoyouyun.com/bee/
![cookies](http://tech.jiu-shu.com/Web-Applications-Technologies/cookie-5.png)
发现还是只有cookie-1和cookie4， 但是其实**cookie2 也被浏览器接受了，只是cookie2 设置的path是/bee/collect 所以基于当前访问路径（http://www.diaoyouyun.com/bee/）chrome的开放工具中无法查看到cookie2。通过查看浏览器上所有站点cookie内容，可以在www.diaoyouyun.com 中找到cookie2。 （**反思：**测试一的 cookie2 是否真的未被接受？）

访问http://diaoyouyun.com/bee/ 查看cookie发现只有cookie-1。（cookie4是子域名下的） 

### 测试三  cookie2 在测试一是否真的未设置成功
cookie2 是path导致的无法查看到? 将path修改后再次走一遍测试一
```
cookie2.setPath("/");
```
![cookie-path](http://tech.jiu-shu.com/Web-Applications-Technologies/cookie-6.png)
经过验证，cookies2未设置成功

### 测试四 跨域请求无法设置cookie
以下请求是无法设置cookie的

![cookie-cross-domain](http://tech.jiu-shu.com/Web-Applications-Technologies/cookie-7.png)
上面的请求cookie1 2 3 4 都无法设置成功。
如果在浏览器直接访问http://api/diaoyouyun.com/bee/cookietest cookie1 和cookie4 可以添加成功
### 总结

 1. 子域名请求可以设置父域名下cookie。即：www.diaoyouyun.com的请求可以设置cookie （domain=diaoyouyun.com，所有diaoyouyun.com及其子域名下site都能查看）
 2. 父级域名的请求**不能设置**子域名的cookie。 即：http://diaoyouyun.com/** 的请求无法设置cookie(domain=www.diaoyouyun.com)。 简化记忆可以理解为，有子肯定存在父，反过来就不成立了。
 3. 跨域的请求是无法成功设置cookie的。



