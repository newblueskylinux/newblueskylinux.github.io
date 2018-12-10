---
layout: post
title:  "solaris10配置ntp服务"
categories: Unix 
tags: solaris ntp
author: 郑禹
---

* content
{:toc}
---
## 一、检查系统软件包
```sh
root@EPPRD1 # pkginfo | grep ntp
system      SUNWntp4r                        NTPv4 (root)
system      SUNWntp4u                        NTPv4 (usr)
system      SUNWntpr                         NTP, (Root)
system      SUNWntpu                         NTP, (Usr)
```





## 二、solaris 10 NTP客户端配置
### 1、创建/etc/inet/ntp.conf 

```sh
#cp -rp /etc/inet/ntp.client /etc/inet/ntp.conf
```

### 2、修改/etc/inet/ntp.conf为如下格式

```sh
vi /etc/inet/ntp.conf
#multicastclient 224.0.1.1  #注释掉这一行
server  10.0.0.65
```
### 3、启动ntp服务并加入开机自启动

```sh
#svcadm enable ntp
```

### 4、查看ntp服务

```sh
# svcs ntp
STATE STIME FMRI
online 9:38:25 svc:/network/ntp:default

# ntpq -p
remote refid st t when poll reach delay offset disp
=============================================================================
*SERVER-IP utcnist2.colora 2 u 63 64 1 0.61 -2.762 15875.0
```

在SERVER-IP前面显示*为客户端同步正常 
无论服务器端还是客户端的ntp -q显示结果，reach值代表是否与服务器端有连接，这个数值一直增大，则基本可以判定连接正常，同时，disp值也会逐渐变小。 
先起服务器端服务，待服务器端确认正常后，再起客户端服务。服务器启动和时间同步都需要比较长的一段时间，请耐心等待

---
