---
layout: post
title:  "jumpserver服务迁移"
categories: Jumpserver 
tags: jumpserver 教程 迁移
author: 郑禹
---

* content
{:toc}
---

## 一、准备需要备份的文件和目录
```sh
/opt/jumpserver/config.yml       #jumpserver系统主配置信息
/opt/start_jms.sh        #启动脚本
/opt/stop_jms.sh         #停止脚本
/opt/jms_docker.sh        #自定义脚本文件，可跳过
/etc/nginx/nginx.conf       #nginx主配置文件
/etc/nginx/conf.d/jumpserver.conf     #jumpserver的nginx配置文件
/etc/nginx/sslkey/1_jumpserver.want-want.com.crt   https证书文件
/etc/nginx/sslkey/2_jumpserver.want-want.key    https私钥文件
/opt/jumpserver/data/media        #此目录下replay目录为录像文件

```
备份数据库文件

```sh
mysqldump -uroot -p jumpserver > /opt/jumpserver.sql
```





## 二、用新版教程配置新服务器并导入数据库文件 (注意 mysql-server 的版本要与旧服务器一致)

修改数据库密码
```sh
mysql -uroot
> grant all on jumpserver.* to 'jumpserver'@'127.0.0.1' identified by '$DB_PASSWORD';  
#修改数据库密码为原系统密码
> flush privileges;
> use jumpserver;
> source /opt/jumpserver.sql;   #这里的数据文件根据实际路径写
> quit
```
## 三、修改/opt/jumpserver/config.yml文件

将下面三个数值修改为原系统的值

SECRET_KEY  BOOTSTRAP_TOKEN  DB_PASSWORD

<img src="http://newbluesky.top/img/jumpmove1.png">

## 四、重启jms进程

此过程如果报错请检查数据库密码和/opt/jumpserver/config.yml 文件里的DB_PASSWD数值是否一致
```sh
/opt/jumpserver/jms stop
/opt/jumpserver/jms start all -d
```
上面一步如果报错请执行下面命令
```sh
python3.6 -m venv /opt/py3
source /opt/py3/bin/activate
```
修改/etc/nginx/nginx.conf和/etc/nginx/conf.d/jumpserver.conf两个文件跟原系统一样，或直接将原系统这两个文件备份出来到新系统里面去替换

## 五、删除docker并新建，跟原系统保持一样

添加koko节点
```sh
docker run --name jms_koko01 -d -p 2222:2222 -p 5000:5000 -e CORE_HOST=http://10.0.110.220:8080 -e BOOTSTRAP_TOKEN=aLoCLqQn829daTny jumpserver/jms_koko:1.5.2
docker run --name jms_koko02 -d -p 2223:2222 -p 5001:5000 -e CORE_HOST=http://10.0.110.220:8080 -e BOOTSTRAP_TOKEN=aLoCLqQn829daTny jumpserver/jms_koko:1.5.2
docker run --name jms_koko03 -d -p 2224:2222 -p 5002:5000 -e CORE_HOST=http://10.0.110.220:8080 -e BOOTSTRAP_TOKEN=aLoCLqQn829daTny jumpserver/jms_koko:1.5.2
docker run --name jms_koko04 -d -p 2225:2222 -p 5003:5000 -e CORE_HOST=http://10.0.110.220:8080 -e BOOTSTRAP_TOKEN=aLoCLqQn829daTny jumpserver/jms_koko:1.5.2
```
添加guacamole节点
```sh
docker run --name jms_guacamole01 -d -p 8081:8081 -e JUMPSERVER_SERVER=http://10.0.110.220:8080 -e BOOTSTRAP_TOKEN=aLoCLqQn829daTny jumpserver/jms_guacamole:1.5.2
docker run --name jms_guacamole02 -d -p 8082:8081 -e JUMPSERVER_SERVER=http://10.0.110.220:8080 -e BOOTSTRAP_TOKEN=aLoCLqQn829daTny jumpserver/jms_guacamole:1.5.2
docker run --name jms_guacamole03 -d -p 8083:8081 -e JUMPSERVER_SERVER=http://10.0.110.220:8080 -e BOOTSTRAP_TOKEN=aLoCLqQn829daTny jumpserver/jms_guacamole:1.5.2
docker run --name jms_guacamole04 -d -p 8084:8081 -e JUMPSERVER_SERVER=http://10.0.110.220:8080 -e BOOTSTRAP_TOKEN=aLoCLqQn829daTny jumpserver/jms_guacamole:1.5.2
```
上面的BOOTSTRAP_TOKEN值根据我们上面的实际值更改，ip地址也修改为自身实际ip地址

## 六、修改启动脚本和停止脚本
```sh
vi /opt/start_jms.sh
#!/bin/bash
set -e

export LANG=zh_CN.UTF-8

systemctl start jms
docker start jms_koko01
docker start jms_guacamole01
docker start jms_koko02
docker start jms_guacamole02
docker start jms_koko03
docker start jms_guacamole03
docker start jms_koko04
docker start jms_guacamole04
exit 0
```
```sh
vi /opt/stop_jms.sh
#!/bin/bash
set -e

export LANG=zh_CN.UTF-8

docker stop jms_koko01
docker stop jms_guacamole01
docker stop jms_koko02
docker stop jms_guacamole02
docker stop jms_koko03
docker stop jms_guacamole03
docker stop jms_koko04
docker stop jms_guacamole04
systemctl stop jms

exit 0
```
## 七、保证节点正常运行

我们加上一个脚本/opt/jms_docker.sh
```sh
vi /opt/jms_docker.sh
#!/bin/bash
date >>/var/log/jumpserver/`date +%F`_jumpserver.log
docker start jms_koko01 >>/var/log/jumpserver/`date +%F`_jumpserver.log
docker start jms_guacamole01 >>/var/log/jumpserver/`date +%F`_jumpserver.log
docker start jms_koko02 >>/var/log/jumpserver/`date +%F`_jumpserver.log
docker start jms_guacamole02 >>/var/log/jumpserver/`date +%F`_jumpserver.log
docker start jms_koko03 >>/var/log/jumpserver/`date +%F`_jumpserver.log
docker start jms_guacamole03 >>/var/log/jumpserver/`date +%F`_jumpserver.log
docker start jms_koko04 >>/var/log/jumpserver/`date +%F`_jumpserver.log
docker start jms_guacamole04 >>/var/log/jumpserver/`date +%F`_jumpserver.log
echo success >>/var/log/jumpserver/`date +%F`_jumpserver.log
exit 0
```
脚本加上可执行权限
```sh
chmod +x /opt/jms_docker.sh
```
将脚本加入到crontab定时任务
```sh
crontab -e
30 * * * * /opt/jms_docker.sh
```
## 八、恢复https的证书文件

如果只放在公司局域网内需要自签名证书，具体过程参照我另一篇教程

## 九、恢复录像文件

[利用openssl创建https证书](http://newbluesky.top/2019/08/10/ssl_self/)

将前面备份的media目录导入到新系统的media目录中即可

* 做到这里，原本数据基本已经恢复，如果发现授权出现在未分组情况可尝试退出session重进或尝试在资产列表重新建一个资产让所有资产数据库刷新

---
