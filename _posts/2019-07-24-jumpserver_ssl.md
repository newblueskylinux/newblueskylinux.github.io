---
layout: post
title:  "jumpserver使用ssl验证网页"
categories: Jumpserver 
tags: jumpserver 教程 ssl
author: 郑禹
---

* content
{:toc}
---


## 修改jumpserver.conf配置文件
```sh
vi /etc/nginx/conf.d/jumpserver.conf
```
<img src="http://newbluesky.top/img/jumpserver_ssl1.png">
修改图示内容为下方内容
```sh
server {
    # 推荐使用 https 访问, 如果不使用 https 请自行注释下面的选项
    listen 443;
    server_name jumpserver.want-want.com;
    ssl on;
    ssl_certificate   /etc/nginx/sslkey/1_jumpserver.want-want.com.crt;
    ssl_certificate_key  /etc/nginx/sslkey/2_jumpserver.want-want.key;
    ssl_session_timeout 5m;
    ssl_ciphers ECDHE-RSA-AES128-GCM-SHA256:ECDHE:ECDH:AES:HIGH:!NULL:!aNULL:!MD5:!ADH:!RC4;
    ssl_protocols TLSv1 TLSv1.1 TLSv1.2;
    ssl_prefer_server_ciphers on;
```
<img src="http://newbluesky.top/img/jumpserver_ssl2.png">
<font size="2.5" color="red">修改完成之后内容如图所示</font>
<br />
这里的crt文件和key文件可向ssl证书申请机构申请购买，如果只放在公司局域网内需要自签名证书，具体过程参照我另一篇博文，[利用openssl创建https证书](http://newbluesky.top/2019/08/10/ssl_self/)

重新加载nginx配置文件或重启nginx生效
```sh
systemctl reload nginx 
或systemctl restart nginx
```
此时会直接访问https://jumpserver-want-want.com虽然可以访问，但如果我们前缀没有加上https就会出现404网页错误，影响用户体验

此时我们可以利用的nginx的重定向功能，将80端口的流量访问重定向到443端口

## 修改nginx主配置文件
```sh
vi /etc/nginx/nginx.conf
```
在图示内容下方加入下面一段代码
```sh
server {
        listen       80;
                server_name jumpserver.want-want.com;
                rewrite ^ https://$http_host$request_uri? permanent;
    }
```
<img src="http://newbluesky.top/img/jumpserver_ssl3.png">
<img src="http://newbluesky.top/img/jumpserver_ssl4.png">
<font size="2.5" color="red">修改完成之后内容如图所示</font>
<br />
重新加载nginx配置文件或重启nginx生效
```sh
systemctl reload nginx 
或systemctl restart nginx
```

---
