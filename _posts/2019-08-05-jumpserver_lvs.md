---
layout: post
title:  "jumpserver多组件负载模式"
categories: Jumpserver 
tags: jumpserver 教程 lvs
author: 郑禹
---

* content
{:toc}
---


## 开启jumpserver网页luna多组件负载

### 一、创建Docker

查看当前运行的docker

```sh
docker ps
```
<img src="http://newbluesky.top/img/jumpserver_lvs1.png">
<font size="2.5" color="red">第一列内容为docker的ID</font>
<br />
如果有未运行的docker，需要加上-a参数
```sh
docker ps -a
```
### 二、删除原本的docker
```sh
docker rm dockerID -f    #加上-f参数可删除正在运行的docker
```
### 三、添加多节点创建docker

创建koko
```sh
docker run --name jms_koko01 -d -p 2222:2222 -p 5000:5000 -e CORE_HOST=http://$Server_IP:8080 -e BOOTSTRAP_TOKEN=$BOOTSTRAP_TOKEN jumpserver/jms_koko:1.5.2
docker run --name jms_koko02 -d -p 2223:2222 -p 5001:5000 -e CORE_HOST=http://$Server_IP:8080 -e BOOTSTRAP_TOKEN=$BOOTSTRAP_TOKEN jumpserver/jms_koko:1.5.2
docker run --name jms_koko03 -d -p 2224:2222 -p 5002:5000 -e CORE_HOST=http://$Server_IP:8080 -e BOOTSTRAP_TOKEN=$BOOTSTRAP_TOKEN jumpserver/jms_koko:1.5.2
```
创建guacamole
```sh
docker run --name jms_guacamole01 -d -p 8081:8081 -e JUMPSERVER_SERVER=http://$Server_IP:8080 -e BOOTSTRAP_TOKEN=$BOOTSTRAP_TOKEN jumpserver/jms_guacamole:1.5.2
docker run --name jms_guacamole02 -d -p 8082:8081 -e JUMPSERVER_SERVER=http://$Server_IP:8080 -e BOOTSTRAP_TOKEN=$BOOTSTRAP_TOKEN jumpserver/jms_guacamole:1.5.2
docker run --name jms_guacamole03 -d -p 8083:8081 -e JUMPSERVER_SERVER=http://$Server_IP:8080 -e BOOTSTRAP_TOKEN=$BOOTSTRAP_TOKEN jumpserver/jms_guacamole:1.5.2
```
四、修改jumpserver.conf配置文件
```sh
vi /etc/nginx/conf.d/jumpserver.conf
```
在图示这部分内容的上方添加下面一段代码

<img src="http://newbluesky.top/img/jumpserver_lvs2.png">
```sh
upstream jumpserver {
    server localhost:8080;
    # 这里是 jumpserver 的后端ip
}

upstream kokows {
    server localhost:5000 weight=1;
    server localhost:5001 weight=1;  # 多节点
    server localhost:5002 weight=1;  # 多节点
    # 这里是 koko ws 的后端ip
    ip_hash;
}

upstream guacamole {
    server localhost:8081 weight=1;
    server localhost:8082 weight=1;  # 多节点
    server localhost:8083 weight=1;  # 多节点
    # 这里是 guacamole 的后端ip
    ip_hash;
}
```
修改图片中的方内容
<img src="http://newbluesky.top/img/jumpserver_lvs3.png">
或使用以下命令修改
```sh
sed -i s/"http://localhost:5000/socket.io/;"/"http://kokows/socket.io/;"/g /etc/nginx/conf.d/jumpserver.conf
sed -i s/"http://localhost:5000/coco/;"/"http://localhost:5000/coco/;"/g /etc/nginx/conf.d/jumpserver.conf
sed -i s/"http://localhost:8081/;"/"http://guacamole/;"/g /etc/nginx/conf.d/jumpserver.conf
sed -i s/"http://localhost:8080;"/"http://jumpserver;"/g /etc/nginx/conf.d/jumpserver.conf
```
重新加载nginx配置文件或重启nginx生效
```sh
systemctl reload nginx 
或systemctl restart nginx
```




## 开启ssh多组件负载

要开启ssh多组件负载同样需要创建docker添加多个koko节点，如果在上面已经做了添加动作，这里就不需要额外操作

只需要修改nginx主配置文件
```sh
vi /etc/nginx/nginx.conf
```
<img src="http://newbluesky.top/img/jumpserver_lvs4.png">
在图示这部分内容下添加下面一段代码
```sh
# 加入 tcp 代理
stream {
    log_format  proxy  '$remote_addr [$time_local] '
                       '$protocol $status $bytes_sent $bytes_received '
                       '$session_time "$upstream_addr" '
                       '"$upstream_bytes_sent" "$upstream_bytes_received" "$upstream_connect_time"';

    access_log /var/log/nginx/tcp-access.log  proxy;
    open_log_file_cache off;

    upstream kokossh {
        server localhost:2222 weight=1;
        server localhost:2223 weight=1;  # 多节点
        server localhost:2224 weight=1;  # 多节点
        # 这里是 koko ssh 的后端ip
        hash $remote_addr;
    }
    server {
        listen 2220;  # 不能使用已经使用的端口, 自行修改, 用户ssh登录时的端口
        proxy_pass kokossh;
        proxy_connect_timeout 10s;
    }
}
# 到此结束
```
在登录的时候只需要输入以下命令
```sh
ssh -p2220 admin@$Server_IP
```
即可使用admin登录jumpserver

添加防火墙规则
```sh
firewall-cmd --zone=public --add-port=2220/tcp --permanent
firewall-cmd --reload
```
重新加载nginx配置文件或重启nginx生效
```sh
systemctl reload nginx 
systemctl restart nginx
```

---
