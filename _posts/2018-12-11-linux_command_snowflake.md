---
layout: post
title:  "Linux命令行实现下雪特效"
categories: Linux
tags: boxes snowflake
author: 郑禹
---

* content
{:toc}
---
## 我们先来看看效果

首先输出这个效果我们要知道cal这个命令，cal是单词calendar的缩写，在linux中视输出日历的命令
[root@linuxstudy ~]# cal
    December 2018
Su Mo Tu We Th Fr Sa
                   1
 2  3  4  5  6  7  8
 9 10 11 12 13 14 15
16 17 18 19 20 21 22
23 24 25 26 27 28 29
30 31
该命令直接在命令行使用将显示本月公历日历并高亮当天的日期
然后需要用到boxes这个命令将日历加"壳"
```sh
[root@linuxstudy ~]# cal|boxes -d dog -p at1l7|boxes -d columns
 __^__                                           __^__
( ___ )-----------------------------------------( ___ )
 | / |           __   _,--="=--,_   __           | \ |
 | / |          /  \."    .-.    "./  \          | \ |
 | / |         /  ,/  _   : :   _  \/` \         | \ |
 | / |         \  `| /o\  :_:  /o\ |\__/         | \ |
 | / |          `-'| :="~` _ `~"=: |             | \ |
 | / |             \`     (_)     `/             | \ |
 | / |      .-"-.   \      |      /   .-"-.      | \ |
 | / | .---{     }--|  /,.-'-.,\  |--{     }---. | \ |
 | / |  )  (_)_)_)  \_/`~-===-~`\_/  (_(_(_)  (  | \ |
 | / | (                                       ) | \ |
 | / |  )            December 2018            (  | \ |
 | / | (         Su Mo Tu We Th Fr Sa          ) | \ |
 | / |  )                           1         (  | \ |
 | / | (          2  3  4  5  6  7  8          ) | \ |
 | / |  )         9 10 11 12 13 14 15         (  | \ |
 | / | (         16 17 18 19 20 21 22          ) | \ |
 | / |  )        23 24 25 26 27 28 29         (  | \ |
 | / | (         30 31                         ) | \ |
 | / |  )                                     (  | \ |
 |___| '---------------------------------------' |___|
(_____)-----------------------------------------(_____)
```

但是我们今天的重点不在这，相信大家也看到了屏幕中的"雪花"才是本次博文的主角

## 技术实现原理

我们知道在unicode编码中有很多特殊字符，其中"雪花"的unicode编码就是u2744，别问我为什么是这个，我也不知道，想知道为什么的可以查看我另一片博文 特殊符号Unicode编码大全

### 取关键参数

此时我们要取三个关键参数，即你当前屏幕行值、列值、小于列值的随机数、"雪花"图形

```sh
[root@linuxstudy ~]# echo $LINES $COLUMNS $(($RANDOM%$COLUMNS)) $(printf "\u2744\n")
40 134 121 ❄
```
我们现在想实现的是让雪花随机出现在这个38*132的屏幕内，并"落"到屏幕最下面一行，那么我们实现的逻辑是在屏幕内设定一个坐标系

![img](G:/%E6%9C%89%E9%81%93%E4%BA%91%E7%AC%94%E8%AE%B0/zhengyulove1234@163.com/0e9c7158c86047908f5d40d1c4549c5d/clipboard.png) 


确定确定"雪花"出现的坐标，并让其横坐标不变的情况下，纵坐标每过一段时间+1，然后"消除"掉原本位置的"雪花"

这就需要构造一个循环数列
```sh
[root@linuxstudy ~]#while true;do echo $LINES $COLUMNS $(($RANDOM%$COLUMNS)) $(printf "\u2744\n");sleep 2;done|awk '{a[$3]=0;for(x in a) {y=a[x];a[x]=a[x]+1;printf "%s;%s ",y,x;printf "%s;%s;%s 0;0\n",a[x],x,$4;}}'
```
![img](G:/%E6%9C%89%E9%81%93%E4%BA%91%E7%AC%94%E8%AE%B0/zhengyulove1234@163.com/9c40fd833fa84dad91edd3d8addce8d2/444.gif) 

可以看到连续两次的输出坐标就是刷新"雪花"和消除“雪花”的坐标

将循环数列的数值转换成坐标

```sh
[root@linuxstudy ~]# while true;do echo $LINES $COLUMNS $(($RANDOM%$COLUMNS)) $(printf "\u2744\n");sleep 2;done|awk '{a[$3]=0;for(x in a) {y=a[x];a[x]=a[x]+1;printf "\033[%s;%sH ",y,x;printf "\033[%s;%sH%s \033[0;0H",a[x],x,$4;}}'
```
为了让"雪花"落得快一些，我们将sleep时间缩短到0.1秒

![1544545541194](C:\Users\zy544\AppData\Local\Temp\1544545541194.png)

下面给出此程序完整的代码

```sh
[root@linuxstudy ~]# clear;printf "\n\n\n\n\n\n";cal|boxes -d dog -p at1l7|awk '{print "                                  "$0}'|boxes -d columns|lolcat;sleep 2;while true;do echo $LINES $COLUMNS $(($RANDOM%$COLUMNS)) $(printf "\u2744\n");sleep 0.1;done|awk '{a[$3]=0;for(x in a) {y=a[x];a[x]=a[x]+1;printf "\033[%s;%sH ",y,x;printf "\033[%s;%sH%s \033[0;0H",a[x],x,$4;}}'
```
在上面的代码中，我为了美观，让这只"狗"跑到了屏幕中央，如果可以的话，加入一些用ANSI控制终端的代码可以让"雪花"也变成彩色，再输入前面加上\033[033m让雪花变成黄色
```sh
[root@linuxstudy ~]# clear;printf "\n\n\n\n\n\n";cal|boxes -d dog -p at1l7|awk '{print "                                  "$0}'|boxes -d columns|lolcat;sleep 2;while true;do echo $LINES $COLUMNS $(($RANDOM%$COLUMNS)) $(printf "\u2744\n");sleep 0.1;done|awk '{a[$3]=0;for(x in a) {y=a[x];a[x]=a[x]+1;printf "\033[%s;%sH ",y,x;printf "\033[%s;%sH\033[033m%s \033[0;0H",a[x],x,$4;}}'
```
很多其他的效果我没有做演示，有兴趣的可以自己尝试

## 总结

上面的代码需要用的命令需要cal,boxes,lolcat,其中cal命令系统存在系统基础包里，其他命令如果在使用中出现command not found则需要安装

boxes一般需要使用epel源yum安装
rhel6：rpm -ivh  http://mirrors.aliyun.com/epel/6/x86_64/Packages/e/epel-release-6-8.noarch.rpm 
rhel7：rpm -ivh  http://mirrors.aliyun.com/epel/7/x86_64/Packages/e/epel-release-7-11.noarch.rpm
lolcat则需要先安装ruby后使用gem安装
