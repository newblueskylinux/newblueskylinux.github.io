---
layout: post
title:  "redhat和centos配置双网卡链路聚合"
categories: Linux
tags: network team
author: 郑禹
---

* content
{:toc}
---
# 图形化模式配置，以centos7.5为例

## 1.在命令行界面输入nm-connection-editor，弹出网络管理界面，点击“+”按钮添加一个网络，选择Team

![img](http://t1.aixinxi.net/o_1cvl7si9e14ra11sh2qo4sr1oufa.png-w.jpg)





## 2.在General栏将开机自动启动勾选上，如下图

![img](http://t1.aixinxi.net/o_1cvl7ttk7rca1fuq1hm1q2c1jn4a.png-w.jpg)

## 3.在Team栏点击“Add”，添加一个“Ethernet”，Interface name 默认为team0

![img](http://t1.aixinxi.net/o_1cvl7v0cnnjric81ucogtancla.png-w.jpg)

## 4.同样在General栏将开机自动启动勾选上，并在Ethernet栏Device选项选择第一块网卡，点击“Save”保存

![img](http://t1.aixinxi.net/o_1cvl806qd199s1clf7cm1qmc1egca.png-w.jpg)

## 5.添加第二块网卡操作同理

## 6.网卡添加完毕后点击“Advanced”按钮打开高级设置

![img](http://t1.aixinxi.net/o_1cvl814to1a5fvn0f4m22t109a.png-w.jpg)

## 7.在打开的选项中Runner栏选择网卡模式，这里我们选“Active backup”，点击“OK”

![img](http://t1.aixinxi.net/o_1cvl82fnu358vh0eplvm21clda.png-w.jpg)



​	这一步在centos7.3以下系统中需要输入配置信息，此配置信息可通过man teamd.conf 查询examples得到："runner": {"name": "activebackup"}

![img](http://t1.aixinxi.net/o_1cvl83bjgs0j1jsp1vo2p6lpo7a.png-w.jpg)

## 8.在命令行输入systemctl restart network 重启网络

## 9.检查配置是否成功，输入命令teamdctl team0 state检查

出现以下信息表示配置成功，主备模式下目前使用的网卡是ens32，若ens32宕机则ens34自动启动维持网络通畅

![img](http://t1.aixinxi.net/o_1cvl8484h11vepvq1i125i1l9fa.png-w.jpg)

# 命令行nmcli命令配置
## 查询帮助man nmcli-examples
查询示例并搜索example 7
```sh
man nmcli-examples
```
```sh
Example 7. Adding a team master and two slave connection profiles
$ nmcli con add type team con-name Team1 ifname Team1 config team1-master-json.conf
$ nmcli con add type team-slave con-name Team1-slave1 ifname em1 master Team1
$ nmcli con add type team-slave con-name Team1-slave2 ifname em2 master Team1
```
## 实际操作命令
### 1.nmcli添加网卡配置
```sh
nmcli con add type team con-name Team0 ifname team0 config '{"runner": {"name": "activebackup"}}'
nmcli con add type ethernet con-name team0-slave1 ifname ens32 master team0
nmcli con add type ethernet con-name team0-slave2 ifname ens34 master team0
```

### 2.修改网卡配置文件
```sh
vi ifcfg-Team0
```
![1545823320778](http://t1.aixinxi.net/o_1cvl85bdpatg2iv1n911267i9ua.png-w.jpg)

将ens32和ens34的网卡配置文件中BOOTPROTO=dhcp，onboot=yes
```sh
systemctl restart network        #重启网络配置
```
输入命令teamdctl team0 state检查，出现以下信息表示配置成功，主备模式下目前使用的网卡是ens32，若ens32宕机则自动启用ens34维持网络通畅

![img](http://t1.aixinxi.net/o_1cvl8484h11vepvq1i125i1l9fa.png-w.jpg)

可能问题：命令行在重启网络后会报错，网络仍然会启动，猜测原因是因为输入的Device名称没有自动寻找mac地址
解决办法：重启机器或down掉网口重启、拔网线