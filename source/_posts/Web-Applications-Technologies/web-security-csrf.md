---
title:  同源策略下为什么还需要防御CSRF
description: 同源策略下为什么还需要防御CSRF，这篇文章用来解释这个疑惑，并给出简单通用的应对策略。
...

前面的章节[搞定跨域资源共享 (CORS)](http://tech.jiu-shu.com/Web-Applications-Technologies/cors-solution)中我们提到了浏览器的同源的安全策略，就算允许跨域，也可以限制被允许的域；比如只限制同一个父域下面的子域名来访问资源。 为什么在这样的策略下面，我们依然需要应对CSRF（Cross Site Request Forgery，跨站请求伪造）？

**通过下面的两张图来快速理解CSRF的存在。**

 ![csrf-form.jpg](http://tech.jiu-shu.com/Web-Applications-Technologies/csrf-form.jpg)
 
 > CSRF攻击的一个很好的例子可能是受害者最近在他们的银行登录帐户并且有效会话启动并运行的攻击。在登录时，会向用户发送一封假冒电子邮件作为他的银行。单击该电子邮件会将他发送到攻击者的站点，该站点的外观可能看起来像银行站点。实际上，该网站对真实银行的运营终端执行POST，以便汇款，将资金从受害者帐户转移到攻击者手中。由于用户的会话仍然有效，因此接受操作并且攻击成功。
 
 ![csrf.png](http://tech.jiu-shu.com/Web-Applications-Technologies/csrf.png)
 
 
 ## 演示CSRF的存在
 我们采用第二章图的方式来演示在同源策略下CSRF的示例。
 
 首先注册并登录： https://www.okchem.com/group-buying
 
 然后打开页面: http://corsweb.jiu-shu.com/csrf-chembnb-logout.html ，此页面会通过以下方式将用户的会话注销。
 ```
 <img style="display:none" src="https://www.okchem.com/group-buying/logout">
 ```
 再次回到https://www.okchem.com/group-buying 刷新页面，可以发现会话已经注销。
 
 >  这里只是个简单的验证，消除疑虑，证明Web网站CRSF防御的存在的必要性。
 
 
## 阻止CSRF策略
**CSRF 的两个重要特点**
- 伪造请求的网站必须利用用户在被攻击的网站的会话
- 伪造请求的网站并没有访问被攻击网站的页面 

**策略一** 从上可以看出，可以在form表单中添加csrf token，服务器端验证比对来防止csrf。

**另外两点和浏览器本身及同源策略有关的是**
- 你在domain a 的页面发送一个请求向domain b的时候可以自动带上domain b的cookie。( 这点给了CSRF的机会)
- 你在domain a的页面无法读取domain b的cookie (这里引出了另一个策略)

**策略二** domain b的交易比对请求头中的csrf token和cookie中的token；b网站只需要在请求的时候从cookie中读取csrf token 放入请求头中个， 在服务器端进行比对即可。

