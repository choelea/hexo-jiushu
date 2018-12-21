---
title: Nginx 问题收集
description: Nginx 使用问题收集
...

收集在使用Nginx过程中遇见的问题。

# 知识积累
## 负载均衡
平均负载示例如下; 以下配置必须保证两个实例都正常运行在，因为这个配置并不会failover。
```
upstream backend {
        server 127.0.0.1:8080;
        server 127.0.0.1:8081;
}
server {
        listen 80;
        server_name auth.jiu-shu.com;

        location / {
            proxy_redirect off;
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_pass http://backend;
            client_max_body_size    10m;
        }
    }
```
> 上例中的8080 和 8081 端口都是Spring Boot的app。由于Java 是多线程的程序，在同一个虚拟机上运行多个实例并非最佳实践；这里只是方便测试。

# 问题收集

### 反向代理后request的host和schema和浏览器请求不一致
反向代理后
下面如果不加proxy_set_header的两行，那么在microservice这个服务中，`request.getScheme() + "://" + request.getServerName()` 就会变成http://microservice.dev.com, nginx rewrite 之后，就可以获取到：http://www.dev.com
```
server_name  www.dev.com;
location / {
		 proxy_set_header Host $host;
  		 proxy_set_header X-Scheme $scheme;	
		 proxy_pass   http://microservice.dev.com:8091;
  	}
```

### bind() to 0.0.0.0:80 failed (98: Address already in use)
启动碰见以上问题，有两种可能

 1. 先检查80端口是否已经被其他http server占用 `sudo netstat -nlpt`
 2. remove the IPv6 bind block (something along the lines of ::1:80。 参考：http://serverfault.com/questions/520535/nginx-is-still-on-port-80-bind-to-0-0-0-080-failed-98-address-already-in
 
### 403 forbidden (13: Permission denied)
参考：[Nginx报错403 forbidden (13: Permission denied)的解决办法](https://www.hi-docs.com/article/detail-MTE1.html)
解决办法一： 关闭 SELinux  （在了解了SELinux的重要性后，决定继续寻找更好的解决办法）

需要进一步了解SELinux相关，需要解决办法二：（感谢Zeal老师给出的解决方案）
> Every directory has a SeLinux context and the default 'Document Root' ( /var/www/html ) has an context which allows the nginx / apache user to access the directory.
The new ROOT ( /data/images ) will not have the same context and thus SeLinux  is blocking the access.
You can verify with ls -lZ /Default-Document-Root and verify the context and associate the same context to /data/images.
This should ideally solve the issue, can you try and verify once  :- 
`chcon -R -u system_u -t httpd_sys_content_t /data/`

相信ftp等服务，如果更改了根目录，也会有同样的问题。需要更深入的对SELinux学习。

#### (13: Permission denied) while connecting to upstream:[nginx]

https://stackoverflow.com/questions/23948527/13-permission-denied-while-connecting-to-upstreamnginx