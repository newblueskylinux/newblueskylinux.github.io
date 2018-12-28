---
layout: post
title:  "Linux配置PPTP协议VPN"
categories: Linux
tags: VPN PPTP
author: 郑禹
---

* content
{:toc}
---
将下面代码保存为vpn_install.sh文件并执行chmod +x vpn_install.sh && ./vpn_install.sh 即可实现一键配置PPTPVPN
```sh
#!/bin/bash
#
# Author:  zhengyu <zy544725571@outlook.com>
# Blog:  https://newbluesky.top
#
# Installs a PPTP VPN-only system for CentOS

# Check if user is root
[ $(id -u) != "0" ] && { echo -e "\033[31mError: You must be root to run this script\033[0m"; exit 1; }

export PATH=/usr/local/sbin:/usr/local/bin:/sbin:/bin:/usr/sbin:/usr/bin
clear
printf "
#######################################################################
#    LNMP/LAMP/LANMP for CentOS/RadHat 5+ Debian 6+ and Ubuntu 12+    #
#            Installs a PPTP VPN-only system for CentOS               #
# For more information please visit https://blog.linuxeye.com/31.html #
#######################################################################
"

#获取公网地址并设置VPN拨号网段IP
[ ! -e '/usr/bin/curl' ] && yum -y install curl
VPN_IP=`curl ipv4.icanhazip.com`
VPN_USER="linuxeye"
VPN_PASS="linuxeye"
VPN_LOCAL="192.168.0.150"
VPN_REMOTE="192.168.0.151-200"

#定义设置VPN的账户密码，可作为单独模块添加VPN账号
while :; do echo
    read -p "Please input username: " VPN_USER
    [ -n "$VPN_USER" ] && break
done
while :; do echo
    read -p "Please input password: " VPN_PASS
    [ -n "$VPN_PASS" ] && break
done
clear

#判断系统版本并根据对应版本安装相应软件包，开启端口转发
if [ -f /etc/redhat-release -a -n "`grep ' 7\.' /etc/redhat-release`" ];then
    #CentOS_REL=7
    if [ ! -e /etc/yum.repos.d/epel.repo ];then
        cat > /etc/yum.repos.d/epel.repo << EOF
[epel]
name=Extra Packages for Enterprise Linux 7 - \$basearch
#baseurl=http://download.fedoraproject.org/pub/epel/7/\$basearch
mirrorlist=https://mirrors.fedoraproject.org/metalink?repo=epel-7&arch=\$basearch
failovermethod=priority
enabled=1
gpgcheck=0
EOF
    fi
    for Package in wget make openssl gcc-c++ ppp pptpd iptables iptables-services
    do
        yum -y install $Package
    done
    echo 'net.ipv4.ip_forward = 1' >> /etc/sysctl.conf
elif [ -f /etc/redhat-release -a -n "`grep ' 6\.' /etc/redhat-release`" ];then
    #CentOS_REL=6
    for Package in wget make openssl gcc-c++ iptables ppp
    do
        yum -y install $Package
    done
    sed -i 's@net.ipv4.ip_forward.*@net.ipv4.ip_forward = 1@g' /etc/sysctl.conf
    rpm -Uvh http://poptop.sourceforge.net/yum/stable/rhel6/pptp-release-current.noarch.rpm
    yum -y install pptpd
else
    echo -e "\033[31mDoes not support this OS, Please contact the author! \033[0m"
    exit 1
fi
echo "1" > /proc/sys/net/ipv4/ip_forward
sysctl -p /etc/sysctl.conf

#将VPN拨号IP和账号密码写入到配置文件，配置VPN服务DNS
[ -z "`grep '^localip' /etc/pptpd.conf`" ] && echo "localip $VPN_LOCAL" >> /etc/pptpd.conf # Local IP address of your VPN server
[ -z "`grep '^remoteip' /etc/pptpd.conf`" ] && echo "remoteip $VPN_REMOTE" >> /etc/pptpd.conf # Scope for your home network
[ -z "`grep '^stimeout' /etc/pptpd.conf`" ] && echo "stimeout 172800" >> /etc/pptpd.conf
if [ -z "`grep '^ms-dns' /etc/ppp/options.pptpd`" ];then
     cat >> /etc/ppp/options.pptpd << EOF
ms-dns 223.5.5.5 # Aliyun DNS Primary
ms-dns 114.114.114.114 # 114 DNS Primary
ms-dns 8.8.8.8 # Google DNS Primary
ms-dns 209.244.0.3 # Level3 Primary
ms-dns 208.67.222.222 # OpenDNS Primary
EOF
fi
echo "$VPN_USER pptpd $VPN_PASS *" >> /etc/ppp/chap-secrets

#设置IPTABLES端口开放和转发规则
ETH=`route | grep default | awk '{print $NF}'`
[ -z "`grep '1723 -j ACCEPT' /etc/sysconfig/iptables`" ] && iptables -I INPUT 4 -p tcp -m state --state NEW -m tcp --dport 1723 -j ACCEPT
[ -z "`grep 'gre -j ACCEPT' /etc/sysconfig/iptables`" ] && iptables -I INPUT 5 -p gre -j ACCEPT
iptables -t nat -A POSTROUTING -o $ETH -j MASQUERADE
iptables -I FORWARD -p tcp --syn -i ppp+ -j TCPMSS --set-mss 1356
service iptables save
sed -i 's@^-A INPUT -j REJECT --reject-with icmp-host-prohibited@#-A INPUT -j REJECT --reject-with icmp-host-prohibited@' /etc/sysconfig/iptables
sed -i 's@^-A FORWARD -j REJECT --reject-with icmp-host-prohibited@#-A FORWARD -j REJECT --reject-with icmp-host-prohibited@' /etc/sysconfig/iptables
service iptables restart
chkconfig iptables on

#重启VPN服务并告知反馈用户信息
service pptpd restart
chkconfig pptpd on
clear
echo -e "You can now connect to your VPN via your external IP \033[32m${VPN_IP}\033[0m"
echo -e "Username: \033[32m${VPN_USER}\033[0m"
echo -e "Password: \033[32m${VPN_PASS}\033[0m"
```