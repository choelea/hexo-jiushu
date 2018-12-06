---
title: kibana的访问控制 - Nginx 反向代理 - 免费
description: 通过Nginx的反向代理来加强kibana的访问安全
---
前一篇[ Kibana 5.x 加强安全](http://blog.csdn.net/choelea/article/details/53841218) 采用的是官方的x-pack 插件来实现elastic技术栈的相关产品的权限控制。功能不错，也提供了很大的灵活性，不过x-pack并非免费产品；咨询了下licence价格，大概三个节点年费六千多美刀。。。废话不多说了，想想替代方案 - Nginx 反向代理 （收回5601端口，通过nginx反向代理+basic authentication来保证安全）

### 参考：
[How To Create a Self-Signed SSL Certificate for Nginx on CentOS 7](https://www.digitalocean.com/community/tutorials/how-to-create-a-self-signed-ssl-certificate-for-nginx-on-centos-7)

### 配置Nginx SSL
#### 第一步: 安装 Nginx 并配置防火墙
参考上面的文章
> **注意： 80 和 443 端口必须对外打开。 当遇见ERR_CONNECTION_REFUSED 这类错误的时候，一定要提高警惕查看端口是否打开。以免浪费时间在配置上面。可以ssh到nginx机器上通过curl 的命令来验证，如果服务器上curl可以访问，外面不可访问；那么很可能端口没开放**

#### 第二步：生成证书
参考上面的文章
#### 第三步：添加kibana.https.conf配置

配置如下：

```
server {
    listen 443 http2 ssl;
    listen [::]:443 http2 ssl;

    server_name kibana.domain.com;

    ssl_certificate /etc/ssl/certs/nginx-selfsigned.crt;
    ssl_certificate_key /etc/ssl/private/nginx-selfsigned.key;
    ssl_dhparam /etc/ssl/certs/dhparam.pem;

    ########################################################################
    # from https://cipherli.st/                                            #
    # and https://raymii.org/s/tutorials/Strong_SSL_Security_On_nginx.html #
    ########################################################################

    ssl_protocols TLSv1 TLSv1.1 TLSv1.2;
    ssl_prefer_server_ciphers on;
    ssl_ciphers "EECDH+AESGCM:EDH+AESGCM:AES256+EECDH:AES256+EDH";
    ssl_ecdh_curve secp384r1;
    ssl_session_cache shared:SSL:10m;
    ssl_session_tickets off;
    ssl_stapling on;
    ssl_stapling_verify on;
    resolver 8.8.8.8 8.8.4.4 valid=300s;
    resolver_timeout 5s;
    # Disable preloading HSTS for now.  You can use the commented out header line that includes
    # the "preload" directive if you understand the implications.
    #add_header Strict-Transport-Security "max-age=63072000; includeSubdomains; preload";
    add_header Strict-Transport-Security "max-age=63072000; includeSubdomains";
    add_header X-Frame-Options DENY;
    add_header X-Content-Type-Options nosniff;

    ##################################
    # END https://cipherli.st/ BLOCK #
    ##################################
    
    # 这里是反向代理到kibana服务 走http协议
    location / {
       proxy_pass   http://localhost:5601;       
    }
}

```
#### 第四步：验证SSL 访问
为设置http跳转的时候，注意在浏览器地址栏中输入https://kibana.domain.com 来验证
#### 第五步： 添加Nginx的Basic Authentication 访问控制

 1. 查看是否有安装httpd-tools `sudo rpm -qa | grep httpd-tools`, 如果有，则可以看到如下信息：`httpd-tools-2.4.6-40.el7.centos.4.x86_64` 如果没有安装，可以通过`sudo yum -y install httpd-tools` 来安装
 2. 配置nginx 反向代理 添加
```
 auth_basic " Basic Authentication ";      
 auth_basic_user_file "/etc/nginx/.htpasswd";
```
 添加至反向代理的配置
```
......
location / {
    proxy_pass   http://localhost:5601;
    auth_basic " Basic Authentication ";      
    auth_basic_user_file "/etc/nginx/.htpasswd";       
}
.....
```
 3. 生成密码文件 `sudo htpasswd -c /etc/nginx/.htpasswd username` 根据提示输入密码
 4. 重新加载ngixn `sudo service nginx reload`
 5. 再次登录来，提示弹出框，输入用户名和密码

