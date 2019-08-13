---
layout: post
title:  "nsenter进入Docker容器"
categories: Jumpserver 
tags: jumpserver 教程 docker
author: 郑禹
---

* content
{:toc}
---


使用nsenter进入Docker容器。关于什么是nsenter请参考如下文章：

[https://github.com/jpetazzo/nsenter](https://github.com/jpetazzo/nsenter)

在了解了什么是nsenter之后，系统默认将我们需要的nsenter安装到主机中

如果没有安装的话，按下面步骤安装即可（注意是主机而非容器或镜像）

具体的安装命令如下：
```sh
wget https://www.kernel.org/pub/linux/utils/util-linux/v2.24/util-linux-2.24.tar.gz  
tar -xzvf util-linux-2.24.tar.gz  
cd util-linux-2.24/  
./configure --without-ncurses  
make nsenter  
sudo cp nsenter /usr/local/bin  
```





在redhat7以后的版本中可以直接使用yum安装
```sh
yum install nsenter
```
安装好nsenter之后可以查看一下该命令的使用

<img src="http://newbluesky.top/img/nsenter1.png">

nsenter可以访问另一个进程的名称空间。所以为了连接到某个容器我们还需要获取该容器的第一个进程的PID。可以使用docker inspect命令来拿到该PID

docker inspect命令使用如下：
```sh
docker inspect --help 
```
inspect命令可以分层级显示一个镜像或容器的信息。比如我们当前有一个正在运行的容器

<img src="http://newbluesky.top/img/nsenter2.png">

可以使用docker inspect来查看该容器的详细信息
```sh
docker inspect fdd3b478896c
```
此处信息太多，只截取一部分展示

<img src="http://newbluesky.top/img/nsenter3.png">

如果要显示该容器第一个进行的PID可以使用如下方式
```sh
docker inspect -f {{.State.Pid}} fdd3b478896c 
```
现在可以看到容器的ID为2571，在拿到该进程PID之后我们就可以使用nsenter命令访问该容器了

<img src="http://newbluesky.top/img/nsenter4.png">

执行下面命令
```sh
nsenter --target 2571 --mount --uts --ipc --net --pid 
nsenter --target 2571 --mount --uts --ipc --net --pid /bin/bash #linux加上/bin/bash
```

如果报错nsenter: failed to execute /bin/bash: No such file or directory 则需要后面加上/bin/su
```sh
nsenter --target 2571 --mount --uts --ipc --net --pid /bin/su
```
这里可以看到已经成功进入容器

<img src="http://newbluesky.top/img/nsenter5.png">

## 脚本方式：
```sh
vi dockerin.sh
#!/bin/bash
pid=`/bin/docker inspect -f {{.State.Pid}} $1`
/bin/nsenter --target $pid --mount --uts --ipc --net --pid
if [ "0" -eq $? ];then
        exit 0
else
/bin/nsenter --target $pid --mount --uts --ipc --net --pid /bin/su
fi
```
* 使用方式
```sh
sh dockerin.sh 容器id
```

---
