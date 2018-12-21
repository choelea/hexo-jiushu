---
title: Centos 7 上利用pm2部署 nodejs 程序 - No Jenkins
description: Centos 7 上利用pm2部署 nodejs 程序 - No Jenkins
---
本文探索如何利用pm2 来部署nodejs的程序到centos系统上。
## 参考资源

 - nodejs的安装 [Installing Node.js via package manager](https://nodejs.org/en/download/package-manager/)
 - npm 权限问题 [Fixing npm permissions](https://docs.npmjs.com/getting-started/fixing-npm-permissions)
 - pm2 deploy [PM2 deployment](http://pm2.keymetrics.io/docs/usage/deployment/)
 - git 安装及版本升级 [How to install GIT on CentOS](https://blacksaildivision.com/git-latest-version-centos)
 - [Deploy and iterate faster, hello ecosystem.json](https://keymetrics.io/2014/06/25/ecosystem-json-deploy-and-iterate-faster/)
## 版本

`git version` 	git version 2.11.1
`node -v` 			v8.0.0
`pm2 -v` 			2.5.0


## 安装nodejs
参考上面的资源链接，在目标服务器上安装nodejs， 本次尝试采用命令：

```
curl --silent --location https://rpm.nodesource.com/setup_8.x | bash -
yum -y install nodejs
```
## 安装pm2

```
npm install pm2@latest -g
```


## 解决SSH用户的权限问题
用root用户显然是不推荐的，然后安装nodejs和npm其他package的安装，我们可能都是通过sudo来安装的。当我们发布程序的时候，我们使用的ssh用户，这里必须解决掉npm perssmision的问题。 参考资源给出了很好的官方的指导；本文采用了第一种方式： `sudo chown -R $(whoami) $(npm config get prefix)/{lib/node_modules,bin,share}` 使用这边命令将owner设置为当前用户。
> 网上的很多文章都忽略了这个问题，这里可以暂时跳过， 到后面的pm2 deploy的时候，问题爆发了再针对性的来解决。


## git 安装
在目标服务器上安装git；（如果通过ssh push的方式，目前机器也可以不安装git）。本次尝试采用在目标服务器上直接pull代码。git 安装参考上面的资源链接。 本文采用了第三方repo来简化安装。（*并非最佳实践，使用第三方的repo需要三思而后行* , 手动安装请参考:[Git Installing from source](https://www.digitalocean.com/community/tutorials/how-to-install-git-on-centos-7)）

```
sudo rpm -U http://opensource.wandisco.com/centos/7/git/x86_64/wandisco-git-release-7-2.noarch.rpm 
sudo yum install -y git
```

> git 版本过低可能引发无法pull到最新的代码的问题。 [Ticket Reference](https://github.com/Unitech/pm2/issues/2436)  CentOS自带的repository只有旧版本和最新版本是1.8.x版本。 

## 示例代码库
示例代码： https://github.com/choelea/image-utils-web.git  branches/pm2-deploy-1

## pm2 发布
参考[官方文档](http://pm2.keymetrics.io/docs/usage/deployment/)， 截止这篇文章的时间，官方文档和代码有些不一致；但是不影响其参考价值。 
### 前提条件

 - 发布机器（可以是你的laptop）可以ssh到目标服务器
 - 目标服务器可以ssh到git拉去代码
 >github上的开源代码不需要ssh可以直接通过https的链接拉去

#### 配置ssh用户
一般的云服务器可以通过web界面的console去配置。 如果没有可以已经有用户名和密码的登录方式，可以通过下面的命令来设置ssh。(后面有示例)

```
$ ssh-keygen -t rsa
$ ssh-copy-id node@myserver.com
```
同样方式设置目标服务器 -> git 服务器的ssh访问。

### 创建deploy的配置文件
通过`pm2 ecosystem` 生成模板进行修改。 可以是json,js,yaml 文件；优选前两种。

```
module.exports = {
  /**
   * Application configuration section
   * http://pm2.keymetrics.io/docs/usage/application-declaration/
   */
  apps : [

    // First application
    {
      name      : 'image-utils-web',
      script    : 'server.js',
      // env: {  If we don't comment here, it will override below deploy config
      //   PORT: 3011
      // },
      env_production : {
        NODE_ENV: 'production'
      }
    }
  ],

  /**
   * Deployment section
   * http://pm2.keymetrics.io/docs/usage/deployment/
   */
  deploy : {
    production : {
      user : 'joe',// ssh 用户名
      host : '192.168.1.188', // 目标服务器地址
      ref  : 'origin/master',
      repo : 'https://github.com/choelea/image-utils-web.git',
      path : '/home/joe/nodejsapp/nodejs-playaround', // 目标服务器部署地址
      'post-deploy' : 'npm install && pm2 reload ecosystem.config.js --env production'
    },
    dev : {
      user : 'osboxes',
      host : '192.168.1.186',
      ref  : 'origin/master',
      repo : 'https://github.com/choelea/image-utils-web.git',
      path : '/home/osboxes/temp',
      'post-deploy' : 'npm install && pm2 reload ecosystem.config.js --env dev',
      env  : {
        NODE_ENV: 'dev',
		PORT: 3012
      }
    }
  }
};
```
运行命令`pm2 deploy ecosystem.config.js dev` 即可进行对应环境的部署。（部署前需要运行`pm2 deploy ecosystem.config.js dev setup` 来创建对应的文件夹。*一般会有current，shared， source 三个文件夹，current是个软连接指向了source*）




## 示例：
### 本地虚拟机测试示例：
如下是利用本地已经生成好的公私钥(id_rsa, id_rsa.pub)，将公钥上传至目标服务器192.168.1.186绑定用户osboxes。
```
$ ssh-copy-id -i id_rsa.pub osboxes@192.168.1.186
/usr/bin/ssh-copy-id: INFO: attempting to log in with the new key(s), to filter                                                             out any that are already installed
/usr/bin/ssh-copy-id: INFO: 1 key(s) remain to be installed -- if you are prompt                                                            ed now it is to install the new keys
osboxes@192.168.1.186's password:

Number of key(s) added: 1

Now try logging into the machine, with:   "ssh 'osboxes@192.168.1.186'"
and check to make sure that only the key(s) you wanted were added.


Administrator@PC-20160321GFRJ MINGW64 ~/.ssh
$ ssh osboxes@192.168.1.186
Last login: Wed Jul  5 1
```
> windows 系统默认没有ssh命令，如果安装了有git，可以通过git bash来运行ssh的命令。

