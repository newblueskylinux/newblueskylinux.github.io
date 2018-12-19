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
命令行nmcli命令配置
man nmcli-examples
1.查询示例，man nmcli-examples ，搜索example 7
Example 7. Adding a team master and two slave connection profiles
$ nmcli con add type team con-name Team1 ifname Team1 config team1-master-json.conf
$ nmcli con add type team-slave con-name Team1-slave1 ifname em1 master Team1
$ nmcli con add type team-slave con-name Team1-slave2 ifname em2 master Team1
2.实际操作命令
nmcli con add type team con-name Team0 ifname team0 config '{"runner": {"name": "activebackup"}}'
nmcli con add type ethernet con-name team0-slave1 ifname ens32 master team0
nmcli con add type ethernet con-name team0-slave2 ifname ens34 master team0
3.正常配置ifcfg-Team0的ip，将ens32和ens34的BOOTPROTO=dhcp，onboot=yes
4.在命令行输入systemctl restart network 重启网络
5.检查配置是否成功，输入命令teamdctl team0 state
问题：（命令行在重启网络后会报错，网络仍然会启动，猜测原因是因为输入的Device名称没有自动寻找mac地址，重启机器或down掉网口重启，拔网线解决过）