---
layout: post
title:  "常用ANSI控制码表"
categories: Linux
tags: ANSI 
author: 郑禹
---

* content
{:toc}
---
##【光标控制】

控制码	说明
[X;YH	将光标移至第 X 行，第 Y 列的位置。
[nA	光标上移 n 列。<<若至屏幕顶端则无效>>
[nB	光标下移 n 列。<<若至屏幕底端则无效>>
[nC	光标向右 n 行。<<若已至屏幕最右端则无效>>
[nD	光标向左 n 行。<<若已至屏幕最左端则无效>>
[6n	报告光标位置。
[s	保存目前的光标位置。
[u	取出保存的光标位置来使用。
[?25l	隐藏光标
[?25h	显示光标





##【屏幕控制】

控制码	说明
[2J	清除屏幕。 <<功能和 DOS 的 CLS 命令一样>>
[K	将光标位置开始至本列结束的所有字符清除。


【显示字符属性控制】

控制码	说明
[n1;n2;......m	设定显示字符的属性状态。
若有两个以上设定则以分号将代码(n1;n2;..)隔开
除非重新设定，否则原设定之属性一直被保留。


代码	意义
0	所有属性 OFF，即返回正常显示模式 (Normal)
1	高亮度显示 (Bright)
4	加下划线 (Underline) (注1)
5	闪烁显示 (Flash) (注2)
7	反相显示 (Inverse)
8	不可见 (Invisable)

前景	背景	颜色
30	40	黑色
31	41	红色
32	42	绿色
33	43	黄色
34	44	蓝色
35	45	紫红色
36	46	青蓝色
37	47	白色

##【字体控制】

控制码	说明
[0I	恢复正常字体
[1I	宋体
[2I	黑体
[3I	楷体





###一、 锁\开键盘代码

　　 代码定义：
　　　　 锁键盘：ESC[2h
　　　　 开键盘：ESC[2l
　　 应用：
　　 1.直接在键盘上顺序敲入Esc＋[＋2＋h四键，键盘即被锁住。
　　 2.在C语言程序显示字符串中包含锁代码：printf(″％c[2h字串″,'\033')。
　　 3.在shell程序中用echo命令：echo″^[[2h字串″.注Esc键在vi编辑器中输入方法为：Ctrl＋v,然后按一下Esc键，在屏幕上显示^[表示已输入。
　　 同样显示开锁代码则键盘被解锁。

###二、 删除字符代码

　　 代码定义：
　　 ESC[nX:清除光标右边n个字符，光标不动。
　　 ESC[K或ESC[OK;清除光标右边全部字符，光标不动。
　　 ESC[1K:清除光标左边全部字符，光标不动。
　　 ESC[2K:清除整行，光标不动。
　　 ESC[J或ESC[OJ:清除光标右下屏所有字符，光标不动。
　　 ESC[1J:清除光标左上屏所有字符，光标不动。
　　 ESC[2J或ESCc:清屏，光标移到左上角。
　　 ESC[nM:删除光标之下n行，剩下行往上移，光标不动。
　　 ESC[nP:删除光标右边n个字符，剩下部分左移，光标不动。
　　 应用：
　　 一般用于自己开发的编辑器中或涉及行编辑的应用程序中。

###三、 插入字符代码

　　 代码定义：
　　 ESC[n@:在当前光标处插入n个字符。
　　 ESC[nL:在当前光标下插入n行。
　　 应用：
　　 一般用于自己开发的编辑器中或涉及行编辑的应用程序中。

###四、 移动光标

　　 代码定义：
　　 ESC[nA:光标上移n行。
　　 ESC[nB:光标下移n行。
　　 ESC[nC:光标右移n个字符。
　　 ESC[nD:光标左移n个字符。
　　 ESC[n;mH :光标定位到第n行m列(类似代码ESC[n;mf)。
　　 应用：
　　 1.一般用于自己开发的编辑器中或涉及行编辑的应用程序中。
　　 2.用shell编辑的菜单程序中定位光标，如：
　　 echo″^[[10;30H请选择：[ ]^[[9C\c″,则先把光标定位到10行30列，然后显示″请选择[ ]″，最后光标右移9个符定位到中括号内等待用户响应。

###五、 定义字符显示属性代码

　　 代码定义：
　　 ESC[p;p;p；……m 其中属性值p可以是一个或多个，具体定义如下：
　　 p 含义
　　 0 清除所有属性
　　 1 高亮度显示
　　 4 下划线(如果硬件支持的话)
　　 5 闪烁(如果硬件支持的话)
　　 25 无闪烁
　　 7 反场(前背景交换)
　　 27 无反场
　　 8 隐藏(不显示)
　　 10 选择基本字体
　　 11 选择第一替代字体；让ASCII值小于32的字 If processing of a directive fails, no further input is read, and scanf() returns. A failure can be either of the following: input failure, mean ing that input characters were unavailable, or matching failure, meaning that the input was inappropriate (see below).The format strin g consists of a sequence of directives which describe how to process the sequence of input characters. If processing of a directi ve fails, no further input is read, and scanf() returns. A failure can be either of the following: input failure, meaning that inp ut characters were unavailable, or matching failure, meaning that the input was inappropriate (see below).The format string consists o f a sequence of directives which describe how to process the sequence of input characters. If processing of a directive fails, no further input is read, and scanf() returns. A failure can be either of the following: input failure, meaning that input character s were unavailable, or matching failure, meaning that the input was inappropriate (see below).符显示时直接取自ROM芯片内
　　 12 选择第二替代字体；在作为ROM字符显示之前先压缩扩展高位ASCII码值
　　 30 前景黑色
　　 31 前景红色
　　 32 前景绿色
　　 33 前景褐色
　　 34 前景蓝色
　　 35 前景紫色
　　 36 前景蓝绿色
　　 37 前景白色
　　 38 开启下划线标志；白色前景用白色下划线
　　 39 关闭下划线标志
　　 40 背景黑色
　　 41 背景红色
　　 42 背景绿色
　　 43 背景褐色
　　 44 背景蓝色
　　 45 背景紫色
　　 46 背景蓝绿色
　　 47 背景白色
　　 应用：
　　 unix系统提供的setcolor命令是以行为单位改变颜色，在实际运用中有它的局限性，上面提到的控制代码在日常生活中用得最多，尤其适合于彩色弹出式立体菜单的设计。如我设计的中心机房值班程序，由于用了这些控制代码，界面非常友好。

###六、其它代码

　　 代码定义：
　　 ESC7：保存当前光标位置参数及字符属性。
	 ESC8：恢复保存的光标位置参数及字符属性。
　　 \n ：插入换行符
　　 \t ：插入一个Tab键。
　　 \r ：插入回车符。
　　 ^G ：插入响铃符(pc嗽叭鸣叫一次)，在vi中按Ctrl＋g即可输入。
　　 \c ：用于shell程序中，不换行。
　　 应用：
　　 1.用户界面设计。
　　 2.用于中心机房向储蓄柜面发通知的程序中(结合实达终端屏幕保存代码ESC[/n;mr__保存从n行到m行的内容，恢复代码ESC[/11)

###七、应用举例

　　 1.彩色立体菜单(shell语言)：
　　 setcolor－b white
　　 clear
　　 while true
　　 do
　　 echo″
　　 ^[[8;20H^[[34;43m┍—————————┒^[[47m
　　 ^[[9;20H^[[34;43m | 1 日 间 业 务 | ^[[40m ^[[47m
　　 ^[[10;20H^[[34;43m| 2 环 境 维 护 | ^[[40m ^[[47m
　　 ^[[11;20H^[[34;43m| 请 选 择 [ ]| ^[[40m^[[47m
　　 ^[[12;20H^[[34;43m┕　　　 　 ┛^[[40m^[[47m
　　 ^[[13;22H^[[40m　　　　　^[[47m
　　 echo″ ^[[11;40H\c″
　　 read acc
　　 done
　　 2.向储蓄所发通知(C语言)：
　　 ＃include<stdio.h>
　　 main()
　　 {
　　 printf(″％c7％c[/2;4r″,'\033',\033');
　　 printf(″％c[2;5H％c[2K　紧急通知　″,' \033','\033');
　　 printf(″％c[3;5H％c[2K由于线路问题，今天不办新余市通存通兑业务“，'[033'，'\033')；
　　 printf(″％c[4;5H％c[2K　中心机房　1998.11.20％c8″,'\033','033','\033');
　　 fflush(stdout);
　　 sleep(10);
　　 printf “％c[/11″，'\033')；
　　 fflush(stdout);
　　 }
　　 说明：该程序首先保存光标位置，同时保存2至4行内容 ，接着定位光标到2行5列，清除该行原先内容，然后显示“紧急通知”字样，通知显示完后立即恢复光标位置，然后等待10秒钟后恢复屏幕原先内容 
--------------------- 
作者：sb___itfk 
来源：CSDN 
原文：https://blog.csdn.net/sb___itfk/article/details/46534723 
版权声明：本文为博主原创文章，转载请附上博文链接！