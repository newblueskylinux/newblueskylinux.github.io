---
layout: post
title:  "jumpserver搭建流程及防坑指南"
categories: Jumpserver 
tags: jumpserver 教程
author: 郑禹
---

* content
{:toc}
---
## 一、环境准备

* 系统版本：

CentOS7.5（Redhat后面会报不支持，坑啊）

* 网络环境：

因为搭建过程中需要安装巨多包，所以还是老老实实连接公网吧。因为公司测试服务器不通外网，所以在公司内网里搭建了一台常规yum源服务器，准备用内网yum源全装所需要得安装包，后来发现有很多脚本还是要连到公网去下载，实在扛不住了，不然自己一个个去分析脚本，人都累死了。

* 系统环境：

建议关闭seliux，或设置httpd_can_network_connect上下文为允许实在不能关的。防火墙开启80，2222端口，如果后续设置https还需要开启443端口






```sh
firewall-cmd --zone=public --add-port=80/tcp --permanent
firewall-cmd --zone=public --add-port=2222/tcp --permanent
firewall-cmd --permanent --add-rich-rule="rule family="ipv4" source address="172.17.0.0/16" port protocol="tcp" port="8080" accept"
firewall-cmd --reload
setsebool -P httpd_can_network_connect 1
```

## 二、安装所需软件包和准备yum源
* 常规yum源：

建议使用aliyun或者网易163的yum源，否则CentOS的官方源会让你奔溃

```sh
wget -O /etc/yum.repos.d/CentOS-Base.repo http://mirrors.aliyun.com/repo/Centos-7.repo
```
配置好常规yum源之后更新更新系统所有软件包和系统内核到最新
```sh
yum update -y
```
更改当前时区，如果在安装系统时有选择此步可不做
```sh
ln -sf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime
```
安装中文语言包，设置英文环境熟悉的可以不做这一步
```sh
yum -y install kde-l10n-Chinese
localedef -c -f UTF-8 -i zh_CN zh_CN.UTF-8
export LC_ALL=zh_CN.UTF-8
echo 'LANG="zh_CN.UTF-8"' > /etc/locale.conf
```
安装必备软件包、epel源
```sh
yum -y install wget gcc git
yum install -y yum-utils device-mapper-persistent-data lvm2 epel-release
yum makecache fast
```
安装docker源
```sh
yum-config-manager --add-repo http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
rpm --import https://mirrors.aliyun.com/docker-ce/linux/centos/gpg
```
安装nginx源
```sh
echo -e "[nginx-stable]\nname=nginx stable repo\nbaseurl=http://nginx.org/packages/centos/\$releasever/\$basearch/\ngpgcheck=1\nenabled=1\ngpgkey=https://nginx.org/keys/nginx_signing.key" > /etc/yum.repos.d/nginx.repo
rpm --import https://nginx.org/keys/nginx_signing.key
```
安装web组件、数据库和docker
```sh
yum -y install redis mariadb mariadb-devel mariadb-server MariaDB-shared nginx docker-ce
```
这里因为我是用其他机器代理上的外网，可能会提示yum秘钥问题导致polkit-pkla-compat-0.1-4.el7.x86_64.rpm无法安装，可以从其它机器上拷贝过来rpm安装
启动web服务、数据库和docker并加入开机自启
```sh
systemctl enable redis mariadb nginx docker
systemctl start redis mariadb
```

## 三、下载组件
安装python3.6和库文件
```sh
yum -y install python36 python36-devel
```
在/opt目录下建立python3.6的环境变量，以下所有python库的安装都需要此环境变量，如果再次期间ssh重连了，则需要重新执行以下操作
```sh
python3.6 -m venv /opt/py3
```
从github上安装jumpserver源码包和luna包
```sh
cd /opt
git clone --depth=1 https://github.com/jumpserver/jumpserver.git
wget https://demo.jumpserver.org/download/luna/1.5.0/luna.tar.gz; tar xf luna.tar.gz; chown -R root:root luna
```
安装jumpserver所需要的组件，这个组件包最好去查看/opt/jumpserver/requirements/rpm_requirements.txt文件里面的内容
```sh
yum -y install $(cat /opt/jumpserver/requirements/rpm_requirements.txt
```
我的系统文件内容是下面这些

libtiff-devel libjpeg-devel libzip-devel freetype-devel lcms2-devel libwebp-devel tcl-devel tk-devel sshpass openldap-devel mariadb-devel mysql-devel libffi-devel openssh-clients telnet openldap-clients 

根据前面安装过程中的安装成功与否，这个文件内容可能不同，但这些组件一定不能有error，否则会造成后面jumpserver起不来
安装自动化运维工具ansible
```sh
source /opt/py3/bin/activate
pip install --upgrade pip setuptools -i https://mirrors.aliyun.com/pypi/simple/
pip install $(cat /opt/jumpserver/requirements/requirements.txt | grep ansible) -i https://mirrors.aliyun.com/pypi/simple/
```
安装python动态网页支持
```sh
pip install $(cat /opt/jumpserver/requirements/requirements.txt | grep python-gssapi) -i https://mirrors.aliyun.com/pypi/simple/
pip install $(cat /opt/jumpserver/requirements/requirements.txt | grep python-keycloak) -i https://mirrors.aliyun.com/pypi/simple/
pip install -r /opt/jumpserver/requirements/requirements.txt -i https://mirrors.aliyun.com/pypi/simple/
```
* 这一步在我的系统上有两个error：

elasticsearch 6.1.1 has requirement urllib3<1.23,>=1.21.1, but you'll have urllib3 1.25.2 which is incompatible.
django-radius 1.3.3 has requirement future==0.16.0, but you'll have future 0.17.1 which is incompatible. 

使用如下命令重新安装下对应版本就可以了
```sh
pip install future==0.16.0
pip install urllib3==1.22
```
配置docker并启动
```sh
curl -sSL https://get.daocloud.io/daotools/set_mirror.sh | sh -s http://f1361db2.m.daocloud.io
systemctl restart docker
docker pull jumpserver/jms_koko:1.5.0
docker pull jumpserver/jms_guacamole:1.5.0
```
配置nginx文件
```sh
rm -rf /etc/nginx/conf.d/default.conf
wget -O /etc/nginx/conf.d/jumpserver.conf https://demo.jumpserver.org/download/nginx/conf.d/jumpserver.conf
```

## 四、处理配置文件
生成随机密码文件
```sh
source ~/.bashrc
DB_PASSWORD=`cat /dev/urandom | tr -dc A-Za-z0-9 | head -c 24`
SECRET_KEY=`cat /dev/urandom | tr -dc A-Za-z0-9 | head -c 50`
echo "SECRET_KEY=$SECRET_KEY" >> ~/.bashrc
BOOTSTRAP_TOKEN=`cat /dev/urandom | tr -dc A-Za-z0-9 | head -c 16`
echo "BOOTSTRAP_TOKEN=$BOOTSTRAP_TOKEN" >> ~/.bashrc
Server_IP=`ip addr | grep inet | egrep -v '(127.0.0.1|inet6|docker)' | awk '{print $2}' | tr -d "addr:" | head -n 1 | cut -d / -f1`
mysql -uroot -e "create database jumpserver default charset 'utf8';grant all on jumpserver.* to 'jumpserver'@'127.0.0.1' identified by '$DB_PASSWORD';flush privileges;"
```
配置jumpserver头文件
```sh
cp /opt/jumpserver/config_example.yml /opt/jumpserver/config.yml
sed -i "s/SECRET_KEY:/SECRET_KEY: $SECRET_KEY/g" /opt/jumpserver/config.yml
sed -i "s/BOOTSTRAP_TOKEN:/BOOTSTRAP_TOKEN: $BOOTSTRAP_TOKEN/g" /opt/jumpserver/config.yml
sed -i "s/# DEBUG: true/DEBUG: false/g" /opt/jumpserver/config.yml
sed -i "s/# LOG_LEVEL: DEBUG/LOG_LEVEL: ERROR/g" /opt/jumpserver/config.yml
sed -i "s/# SESSION_EXPIRE_AT_BROWSER_CLOSE: false/SESSION_EXPIRE_AT_BROWSER_CLOSE: true/g" /opt/jumpserver/config.yml
sed -i "s/DB_PASSWORD: /DB_PASSWORD: $DB_PASSWORD/g" /opt/jumpserver/config.yml
```

## 五、启动jumpserver
开启web并启动docker应用
```sh
systemctl start nginx
cd /opt/jumpserver
./jms start all -d
docker run --name jms_koko -d -p 2222:2222 -p 5000:5000 -e CORE_HOST=http://$Server_IP:8080 -e BOOTSTRAP_TOKEN=$BOOTSTRAP_TOKEN jumpserver/jms_koko:1.5.0
docker run --name jms_guacamole -d -p 8081:8081 -e JUMPSERVER_SERVER=http://$Server_IP:8080 -e BOOTSTRAP_TOKEN=$BOOTSTRAP_TOKEN jumpserver/jms_guacamole:1.5.0
systemctl restart nginx
```
显示当前配置信息
```sh
echo -e "\033[31m 你的数据库密码是 $DB_PASSWORD \033[0m" \
&& echo -e "\033[31m 你的SECRET_KEY是 $SECRET_KEY \033[0m" \
&& echo -e "\033[31m 你的BOOTSTRAP_TOKEN是 $BOOTSTRAP_TOKEN \033[0m" \
&& echo -e "\033[31m 你的服务器IP是 $Server_IP \033[0m" \
&& echo -e "\033[31m 请打开浏览器访问 http://$Server_IP 用户名:admin 密码:admin \033[0m"
```
## 六、配置jumpserver开机自启
```sh
wget -O /usr/lib/systemd/system/jms.service https://demo.jumpserver.org/download/shell/centos/jms.service
chmod 755 /usr/lib/systemd/system/jms.service
wget -O /opt/start_jms.sh https://demo.jumpserver.org/download/shell/centos/start_jms.sh
wget -O /opt/stop_jms.sh https://demo.jumpserver.org/download/shell/centos/stop_jms.sh
"sh /opt/start_jms.sh" >> /etc/rc.local
chmod +x /etc/rc.d/rc.local
```
启动停止的脚本为/opt/start_jms.sh, 如果自启失败可以手动启动

---
