---
title:  搞定跨域资源共享 (CORS)
description: 通过多个Nodejs Web App一步步来深入了解CORS每个细节，每步都通过实际的验证；最终给出Nodejs Express Web和Java Spring Web的代码示例。
...

通过多个Nodejs Web App一步步来深入了解CORS每个细节，每步都通过实际的验证；最终给出Nodejs Express Web和Java Spring Web的代码示例。

测试CORS代码库： [git@github.com:choelea/cors-tester.git](git@github.com:choelea/cors-tester.git)

![cors](http://tech.jiu-shu.com/Web-Applications-Technologies/cors-font-request.png)
## 什么是CORS
解释这个概念之前先要认识下什么是 域(Origin)。
### 什么是Origin 
域是 协议(http/https)+域名+端口的组合。 `An origin is defined as a combination of URI scheme, host name, and port number. `摘自：https://en.wikipedia.org/wiki/Same-origin_policy

>这个是一个标准，但不是所有浏览器的所有版本都严格执行了，特别是关于端口这点。

通过下面的表可以更直观的认识到什么才是'**同一个域(同源)**'。(图标截自维基百科)
![Same Origin Metric](http://tech.jiu-shu.com/Web-Applications-Technologies/same-origin-table.png)

### CORS 定义
Cross-origin resource sharing (CORS)； 跨域资源共享（CORS）是一种机制，这种机制在允许在网页中请求另一个域**受限制**的资源。
摘自维基百科: https://en.wikipedia.org/wiki/Cross-origin_resource_sharing

> 这里的受限制并不是说这些资源需要登录厚着授权等
## 哪些资源是默认可以跨域的
上面定义提到了"受限制", 也就是说不是所有的跨域资源需要CORS机制。在不做任何设置的的时候，默认图片, css, js 等请求都是可以跨域的。
> 如果你的css有针对字体的请求，你会发现字体请求**默认**也是受到**同源机制**的限制；包括js 和css对应的map文件的请求都无法跨域访问。

#### AJAX请求可以吗
浏览器打开`http://corsdisableapi.jiu-shu.com/users`可以获取到json结果如下：
```
[
    {
        "name": "Joe"
    },
    {
        "name": "Mark"
    }
]
```
但是我们打开页面`http://corsweb.jiu-shu.com/public-resources.html` (或者任何其他域),在console里面做出如下请求：
```
var xhr = new XMLHttpRequest();
xhr.open('GET', 'http://corsdisableapi.jiu-shu.com/users'); 
xhr.send();
```
你会发现console报出了如下的错误; 很明显请求是收到**同源机制**的限制。
![cors policy error](http://tech.jiu-shu.com/Web-Applications-Technologies/cors-policy-error.png)

打开页面http://corsweb.jiu-shu.com/public-resources.html 通过源代码和开发者工具理解这一节知识。
## 开启CORS
很明显很多时候我们必须有个策略来**突破/放宽**同源政策的限制； 比如Web页面www.example.com 需要请求api.example.com的资源；比如：PC站www.example.com 和M站m.example.com 需要共同获取/修改api.example.com的资源。
> CORS 只是**突破/放宽同源政策**中的一个种, 其他具体可以参考： https://en.wikipedia.org/wiki/Same-origin_policy

CORS需要浏览器和服务器同时支持。目前，所有浏览器都支持该功能，IE浏览器不能低于IE10。

整个CORS通信过程，都是浏览器自动完成，不需要用户参与。对于开发者来说，CORS通信与同源的AJAX通信没有差别，代码完全一样。浏览器一旦发现AJAX请求跨源，就会自动添加一些附加的头信息，有时还会多出一次附加的请求，但用户不会有感觉。

因此，实现CORS通信的关键是服务器。只要服务器实现了CORS的标准，就可以跨源通信。


### 服务端开启CORS
这里示例代码均采用nodejs express 的web应用，使用[cors](https://www.npmjs.com/package/cors)组件即可轻松实现服务端开启CORS。
将之前的corsdisableapi.jiu-shu.com服务复制一份增加CORS的支持，以corsenable.jiu-shu.com这个域来提供；然后访问页面http://corsweb.jiu-shu.com/request-cors-resources.html 对比前面一个页面 http://corsweb.jiu-shu.com/public-resources.html 可以发现之前浏览器console抛出的cors相关的错误全部消失了。

## 浏览器的两种CORS请求
大部分有些了解CORS都听过OPTION请求; 也叫"预检"请求（preflight）。但是前面的页面http://corsweb.jiu-shu.com/request-cors-resources.html 中的跨域的请求，通过开发者工具查看，在network这个标签中无法找到OPTION的请求。这是因为CORS有两种请求：简单请求和非简单请求；简单请求是不需要**预检**请求的。 但是无论是什么CORS请求，浏览器都会自动加上Origin这个header。
### 简单请求
一般来说满足下面的有可能是简单请求。
- 请求方法是 HEAD/GET/POST
- HTTP的头信息中没有自定义的Header；Content-Type不能是application/json

简单请求，浏览器只需要发出一个请求就可以拿到想要的结果。
> 暂时没有完全找到简单请求的完整定于及所有场景，但是一般情况我们不需要考虑，因为你一定需要支持非简单请求的情况。
### 非简单请求
非简单请求就需要"预检"请求(preflight); 浏览器根据preflight的结果来决定下一个正式请求**是否可以发**以及**怎么发**。

访问页面http://corsweb.jiu-shu.com/request-cors-resources.html 打开console输入下面请求，观察网络请求，可以发现两个请求。
```
var xhr = new XMLHttpRequest();
xhr.open('GET', 'http://corsenableapi.jiu-shu.com/users'); 
xhr.setRequestHeader('Content-Type', 'application/json');
xhr.send();
```

## CORS相关字段

理解上面之后我们需要开始了解头信息中和CORS相关的字段，这些字段都是Access-Control-开头。
### 预检请求的相关字段
**Access-Control-Request-Method**
该字段是必须的，用来列出浏览器的CORS请求会用到哪些HTTP方法。

**Access-Control-Request-Headers**
该字段是一个逗号分隔的字符串，指定浏览器CORS请求会额外发送的头信息字段。

### 响应的相关字段
**Access-Control-Allow-Origin**
该字段是必须的。它的值要么是请求时Origin字段的值，要么是一个*，表示接受任意域名的请求。

**Access-Control-Allow-Methods**
该字段必需，它的值是逗号分隔的一个字符串，表明服务器支持的所有跨域请求的方法。注意，返回的是所有支持的方法，而不单是浏览器请求的那个方法。这是为了配合Access-Control-Max-Age来避免多次"预检"请求。

**Access-Control-Allow-Headers**
如果浏览器请求包括Access-Control-Request-Headers字段，则Access-Control-Allow-Headers字段是必需的。它也是一个逗号分隔的字符串，表明服务器支持的所有头信息字段，不限于浏览器在"预检"中请求的字段。

**Access-Control-Allow-Credentials**
该字段可选。它的值是一个布尔值，表示是否允许发送Cookie。默认情况下，Cookie不包括在CORS请求之中。设为true，即表示服务器明确许可，Cookie可以包含在请求中，一起发给服务器。这个值也只能设为true，如果服务器不要浏览器发送Cookie，删除该字段即可。
> 这个设置是容易被忽视的，同时也需要前端配合的。一个很容易想到的场景就是Session会话，Session会话往往是以secure的cookie的形式存在，当你的网站有多个子域名，而这些子域名都共享一个会话的时候，你的AJAX的CORS请求就需要带上Cookie。如果前端用的是Jquery，可以参考：https://www.html5rocks.com/en/tutorials/cors/#toc-cors-from-jquery

> 当开启Credentials的时候，为了安全考虑，浏览器要求Access-Control-Allow-Origin 必须制定值不能用`*`，否则会得到如下的错误

![credentials restrict specific origin](http://tech.jiu-shu.com/Web-Applications-Technologies/credentials-restrict-specific-origin.png)

```
var xhr = new XMLHttpRequest();
xhr.open('GET', 'http://corssession.jiu-shu.com/viewhistories'); 
xhr.setRequestHeader('Content-Type', 'application/json');
xhr.withCredentials = true;
xhr.send();
```
**Access-Control-Max-Age**
该字段可选，用来指定本次预检请求的有效期，单位为秒。上面结果中，有效期是20天（1728000秒），即允许缓存该条回应1728000秒（即20天），在此期间，不用发出另一条预检请求。

**Access-Control-Expose-Headers**
该字段可选。CORS请求时，XMLHttpRequest对象的getResponseHeader()方法只能拿到6个基本字段：Cache-Control、Content-Language、Content-Type、Expires、Last-Modified、Pragma。如果想拿到其他字段，就必须在Access-Control-Expose-Headers里面指定。

## 解决CORS问题的两个步骤
- 确定服务端响应字段是否正确
- 如果服务端响应字段正确，确定客户端是否正确发送了请求，需要的header是否都带上了，需要的cookie是否带上了

## 最佳实践
介绍了这么多，是时候来点干货 了。。。

### 好的CORS需求如下
-  只允许制定的域访问 (CORS请求中的Origin如果通过，就原样设置回Access-Control-Allow-Origin)
-  CORS请求需要带上Cookie
-  减少不必要的预检请求 (Access-Control-Max-Age  需要用)

### Nodejs Express Web的代码
```
const cors = require('cors');
...
const corsOptioin = {
  "origin": /\.jiu-shu\.com$/, // jiu-shu.com 的所有子域名
  "methods": "GET,HEAD,PUT,PATCH,POST,DELETE",
  "optionsSuccessStatus": 204,
	"allowedHeaders":"X-Csrf-Token, X-Requested-With", // 给出你允许的所有的Header
  "credentials":true, // 服务端可以让你带上Cookie，如果没带上就去检查你的前端代码
  "maxAge":3600  // 一个小时内预检一次就OK啦
};
app.use(cors(corsOptioin));
```
### Java Spring Web的代码
 非Spring 的也同这个相似。
 
 ```Java
 import java.io.IOException;
import java.util.regex.Pattern;

import javax.servlet.FilterChain;
import javax.servlet.ServletException;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;

import org.springframework.beans.factory.annotation.Value;
import org.springframework.stereotype.Component;
import org.springframework.web.filter.OncePerRequestFilter;
 

@Component
public class CorsFilter extends OncePerRequestFilter {
 
	@Value("${cors.origins.pattern}")
	private String originPattern;  // (.+\\.)*lichao\\.com
	    
    @Override
	protected void doFilterInternal(HttpServletRequest request, HttpServletResponse response, FilterChain filterChain)
			throws ServletException, IOException {		 
        Pattern hostAllowedPattern = Pattern.compile(originPattern, Pattern.CASE_INSENSITIVE);
    	String origin = request.getHeader("Origin");
    	if(origin!=null) {    		
    		if (hostAllowedPattern.matcher(origin).matches()) {
    			response.addHeader("Access-Control-Allow-Origin", origin);// (CORS请求中的Origin如果通过，就原样设置回Access-Control-Allow-Origin)	
    			response.addHeader("Access-Control-Allow-Methods","GET, OPTIONS, HEAD, PUT, POST, DELETE"); 
    			response.setHeader("Access-Control-Allow-Credentials", "true");
    			response.setHeader("Access-Control-Max-Age", "3600");
    			if (request.getMethod().equals("OPTIONS")) {
    				// response.addHeader("Access-Control-Allow-Headers", request.getHeader("Access-Control-Request-Headers"));
						response.addHeader("Access-Control-Allow-Headers", "locale, X-Csrf-Token, X-Requested-With"); // List All
    				response.setStatus(HttpServletResponse.SC_ACCEPTED);
    				return;
    			}
    		} else {
    			// Throw 403 status OR send default allow
    			response.addHeader("Access-Control-Allow-Origin", "https://www.jiu-shu.com");
    		}    		
    	} 
    	filterChain.doFilter(request, response);	
	}    
}
 ```



参考文章: 

https://en.wikipedia.org/wiki/Same-origin_policy

https://en.wikipedia.org/wiki/Cross-origin_resource_sharing

http://www.ruanyifeng.com/blog/2016/04/cors.html

http://www.ruanyifeng.com/blog/2016/04/same-origin-policy.html

https://blog.csdn.net/guodengh/article/details/73187908?locationNum=7&fps=1
