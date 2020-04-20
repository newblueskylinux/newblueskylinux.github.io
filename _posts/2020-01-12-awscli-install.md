---
layout: post
title:  "awscli命令安装及使用"
categories: AWS 
tags: awscli 教程
author: 郑禹
---

* content
{:toc}
---

	云服务器的运维势必会成为未来运维人员不可或缺的技能之一，专注于一两个操作系统做自动化运维的也无法长久
	工欲善其事必先利其器，话不多说，基础乃重中之重，学会怎么安装才能好好玩

## 一、安装 AWS CLI

```sh
pip install awscli
```

Python虚拟环境相关，参考： [https://www.jianshu.com/p/d66fce9a7bdc](https://www.jianshu.com/p/d66fce9a7bdc)





查看当前版本
```sh
aws --version
```
升级到最新版
```sh
aws install awscli --upgrade
```
卸载awscli
```sh
pip uninstall awscli
```
## 二、配置 AWS CLI
添加默认的配置文件
未使用过 AWS CLI，则必须先配置默认的 CLI 配置文件
```sh
aws configure
AWS Access Key ID [None]: XXXXXXXXXXXXX（你的AWS访问KEY）
AWS Secret Access Key [None]: XXXXXXXXXXXXXXXX（你的AWS访问密钥）
Default region name [None]: cn-northwest-1
Default output format [None]: json
```
这一步做完之后会在当前家目录的aws目录下生成两个文件，config和credentials

里面的数据明文存放，在 .aws/config 文件中声明新账号所在区域内容
```sh
[default]
region = cn-northwest-1
output = json
```
在 .aws/credentials 文件中存放访问key和密钥内容
```sh
[default]
aws_access_key_id = XXXXXXXXXXXXX（你的AWS访问KEY）
aws_secret_access_key = XXXXXXXXXXXXXXXX（你的AWS访问密钥）
```

** 到这里awscli工具基本已配置完毕，看完是不是很简单呢。

    当然里面省略了一些内容，对于初学者可能有一些疑问，比如为什么我的linux系统使用pip install安装的时候报错
    Key ID和Access Key从哪里获取，总不可能凭空产生吧。请带着这些疑问移步到我另一篇文章[初识aws控制台]

验证awscli是否配置成功也很简单
```sh
aws s3 ls
```
---
