---
layout: post
title:  "jumpserver的MFA令牌遗失"
categories: Jumpserver 
tags: jumpserver 教程 
author: 郑禹
---

* content
{:toc}
---


## 个人信息开启MFA令牌

如果是普通用户联系管理员关闭MFA, 登录成功后用户在个人信息里面重置MFA令牌

普如果是管理员在个人信息设置里面开启的MFA二次认证，且遗失MFA令牌无法登陆

可通过登录jumpserver操作系统，修改数据库 users_user 表对应用户的 otp_level 为 0

### 关闭MFA功能

下次开启MFA功能时继续使用上次绑定的MFA令牌，admin 为要修改的用户
```sh
mysql -uroot
> use jumpserver;
> update users_user set otp_level='0' where username='admin'; 
```
###通过数据库重置MFA令牌

修改数据库 _otp_secret_key 为 0 ,绑定新的MFA令牌
```sh
mysql -uroot
> use jumpserver;
> update users_user set otp_secret_key='0' where username='admin'; 
```

---
