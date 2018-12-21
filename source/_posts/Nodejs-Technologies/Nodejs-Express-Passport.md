---
title: Nodejs 利用passport完成本地认证 示例
description: 阐述Nodejs + express + passport 完成用户回话的管理
...

本文只涉及web相关，即浏览器作为客户端。一步一步理解认证过程，同时熟悉express-session,passport, connect-flash 各自的职责。示例并没有引入mongodb, 方便更直接的理解认证过程。（In-Memory Authentication Example）
# 示例实现场景介绍

 - 用户访问需要登录的页面，被重定向至登录页面
 - 登录界面输入用户名和密码，服务端验证并返回显示错误信息, 验证成功后返回至用户先前访问的页面
 - 如果用户直接访问的登录页面，登录成功后返回指定页面（这里指定/session-demo 页面方便演示）

# 资源链接
- [express-session](https://github.com/expressjs/session)
- [passport](http://passportjs.org/docs/overview)
- [connect-flash](https://www.npmjs.com/package/connect-flash)

**代码**： https://github.com/choelea/image-utils-web  **releases**： passport-local-authentication
# 逐步探索
探索express + session + passport 来完成用户的基础认证。 网络上大部分文章及示例都是将mongoose，express-session，passport, connect-flash等捆绑在一起，没有一步一步解释清晰各自的职责， 其实这些组件之间并没有多大的依赖关系。
## express-session 登场
有过web开发经验的对session都是有所了解，简单澄清一点：
> session 中的数据存放服务器端，浏览器端的cookie会以某种形式存放session的ID，服务器端获取到这个id可以通过这个id索引到session data。

通过`npm install express-sessioin --save` 添加依赖。代码中找到合适的位置引入依赖：

```
const session = require('express-session');
......
app.use(session({ secret: 'secretkey', resave: false, saveUninitialized: false })); // 参数的说明请参考文档，saveUninitialized 和 resave一般推荐设置为false
```
搞定， 可以再次访问页面，可以通过浏览器开发者工具看到名称为'connect.sid' 的cookie会创建。创建一下的route (/session-demo)来测试：

```
app.use('/session-demo', require('./routes/session-demo'));
```
session-demo.js 主要代码如下：
```
let router = require('express').Router();

router.get('/', (req, res) => {
    let sess = req.session;
    console.log(`session id is: ${sess.id}`)
    res.json(sess);
});
module.exports = router;
```
运行demo进行测试；查看cookie中'connect.sid' 和后端log出来的id不一致，是因为cookie中的id是被加密了的。详情可以查看文档中的option - secret。
> 可以在`/session-demo` 这里添加代码往session里面添加data，刷新这个页面来测试session的效果。默认配置创建session后，在没有往session添加东西之前，session每次都会变化。
 
## passport 入场
添加依赖`npm install passport --save`，`npm install passport-local --save` 
**passport功能单一，只做authenticate**, 但是认证方式多种多样，具体如何认证，passport采用了策略模式把具体的实现留给了具体的strategies; 比如passport-local. 
> 策略模式是一种设计模式，它将算法和对象分离开来，通过加载不同的算法来实现不同的行为，适用于相关类的成员相同但行为不同的场景，比如在passport中，认证所需的字段都是用户名、邮箱、密码等，但认证方法是不同的。
#### 配置passport及strategy
配置passport， 方便起见，创建单独的配置文件：config/passport.js

```
'use strict';

/*!
 * Module dependencies.
 */

const LocalStrategy = require('passport-local').Strategy;
const passport = require('passport');
/**
 * Expose
 */
passport.serializeUser(function(user, done) {
  done(null, user);
});

passport.deserializeUser(function(user, done) {
  done(null, user);
});

// use these strategies
passport.use('local',new LocalStrategy(
    function (username, password, done) {
        console.log(`Trying to verify user, username:${username} password:${password}`)
        if (username != 'joe' || password != 'password') {
            console.log(`Failed to verify user, username:${username} password:${password}`)
            return done(null, false, { message: 'Invalid username or password' });
        }
        return done(null, { "username": username, "password": password },{message:'Successfully authenticated!'});
    }
));
module.exports = passport;
```
> 引用passport官方文档：　In a Connect or Express-based application, `passport.initialize()` middleware is required to initialize Passport. If your application uses persistent login sessions, `passport.session()` middleware must also be used.

这个示例我们不session的持久化，在内存中管理session，因此只需要在相关代码（server.js）加入：

```
const passport = require('./config/passport');
app.use(passport.initialize());
```
> session的持久化在生产环境和重要，这里只是方便演示。

## 登录登出
### 添加login 页面
具体代码参考源代码`routes.js` 和 `login.hbs` ; form表单的post请求为`/login`
### 添加接受/login post 请求
routes.js 中添加：

```
const passport = require('passport')
.......
app.post('/login',
        passport.authenticate('local', {
            successRedirect: '/session-demo',
            failureRedirect: '/login',// 可以先尝试设置/session-demo 来查看对应的情况
            failureFlash: true
        }));
.......
```
> 来自官方文档：　Setting the **failureFlash** option to true instructs Passport to flash an error message using the message given by the strategy's verify callback, if any. 

这里的flash error message来自passport strategy的callback。比如：`return done(null, false, { message: 'Invalid username or password' });` 
> 设置`failureRedirect: '/session-demo'` 可以看看到对应的错误信息。

![express-session-passport-failed](http://tech.jiu-shu.com/Nodejs-Technologies/session-json.jpg)

输入正确用户名和密码：（joe/password）可以在session-demo的页面看到如下：
![express-session-passport-success](http://tech.jiu-shu.com/Nodejs-Technologies/session-json1.jpg)

### connect-flash的作用
又一个从express剥离的组件; 简单读取文档既可以了解到flash 的信息会保存在session中。 
引入此依赖，同时需要在session相关的配置代码下面添加：`app.use(flash());`。 
如果尝试将login失败重定向至session-demo页面，然后多次尝试输入错误的用户名和密码。将会在session-demo页面看到重复的类似如下的flash的error信息：
![express-session-passport-duplicated-errros](http://tech.jiu-shu.com/Nodejs-Technologies/login-error.jpg)
> 这些信息是保存在session中，又没有被消费掉；**必须被消费掉，否则会挤爆内存**，如何消费？看下面说明

### 登录错误显示
**错误信息存放于会话中，并且需要及时消费掉！！！ ** 设置login失败返回login页面，及时消费掉session中的flash信息， 代码如下：

```javascript
app.get('/login', (req, res) => {
        let errors = req.flash('error');  // 消费掉flash中的error信息
        res.render('login', { 'errors': errors })
    });
```
页面使用boostrap显示错误信息，请参考源码`login.hbs`。

### 登出
参考passport文档添加一下logout代码：
```
app.get('/logout', function(req, res){
  req.logout();
  res.redirect('/session-demo');
});
```
可以在response中看到passport信息从session中消失了。
## 会话中用户的序列化和发序列化
截止上面，虽然完成了登录，用户的信息也在session中了，但是代码中如何获取到当前用户呢？
前面的passport.js代码中的`serializeUser` 和 `deserializeUser` 分别是设置和从session中获取用户的时候被调用。
> serializeUser和deserializeUser一般有两种方式：1，整个用户模型都存储于会话，获取时候(deserializeUser)无需查询数据库; 2， 只存储用户的ID，获取的时候根据ID从数据库中查询用户模型。  这里的示例采用的是第一种。 

文档中多次提到了req.user， 做如下尝试：

```
 app.get('/me', (req, res) => {
     console.log(`current user is ${req.user}`)
     res.json({'user':req.user});
 });
```
登录后，访问`/me` 无效。 Why？
**解决方法：** 添加 `app.use(passport.session());`；一定要在`express.session()` 之后添加。
> 前面提到了session的持久化，由于我们并没有做持久化，所以没有引入`passport.session()` 这个中间件。 现在来看， 显然这个中间件并非只和会话的持久化相关。passport的文档并没有详细描述这点。

## 路由权限控制中间件
**问题：**一个web网站的内容往往有公开的和私有的部分，如何配置让私有请求重定向到登录页面，甚至针对角色权限不同显示相应的错误消息？
添加config/auth.js

```
exports.requiresLogin = function (req, res, next) {
  if (req.isAuthenticated()) return next();
  if (req.method == 'GET') req.session.returnTo = req.originalUrl;
  res.redirect('/login');
};
```
在route里面来引用配置

```
const auth = require('./config/auth')

...
app.get('/me', auth.requiresLogin, (req, res) => {
    console.log(`current user is ${req.user}`)
    res.json({ 'user': req.user });
});
...// 顺便重构下POST login 保证成功后跳转至用户之前的GET 请求页面
app.post('/login',
        passport.authenticate('local', {
            failureRedirect: '/login',
            failureFlash: true
        }), (req, res) => {
            const redirectTo = req.session.returnTo? req.session.returnTo: '/session-demo';
            delete req.session.returnTo;
            res.redirect(redirectTo);
        });
```
这里修改了`/me` 请求，使用`auth.requiresLogin` 来强制用户登录。同时注意修改了`POST /login` 登录请求的处理来方便登陆后重定向到之前的`GET 请求页面` 。 再次以未登录的状态访问`/me` 就会被重定向至登录界面，登录回来后直接就跳转至了`/me` 页面展示对应的用户信息。