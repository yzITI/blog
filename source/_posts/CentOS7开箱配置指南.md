---
title: CentOS7开箱配置指南
date: 2019-04-06 12:37:07
categories:
- Linux服务器
tags:
- CentOS
- Linux
- 服务器
toc: true
---

本文记录了一些简明的CentOS7使用方法
<!-- more -->

# SELINUX
> selinux 存在的意义似乎只是提醒我们把它关掉……

关闭方法：修改`/etc/selinux/config`
其中的 enforcing 改为  disabled

# 配置IP
> 由于我门的服务器集群需要手动配置IP，特在此备忘

修改`/etc/sysconfig/network-scripts/`下面的`ifg-enoxxxxxxx`形式的文件，参考下面的模板：
```toml
HWADDR=[不改]
TYPE=Ethernet
BOOTPROTO=none
IPADDR=[要设置的IP]
DEFROUTE=yes
PEERDNS=yes
PEERROUTES=yes
IPV4_FAILURE_FATAL=no
IPV6INIT=yes
IPV6_AUTOCONF=yes
IPV6_DEFROUTE=yes
IPV6_PEERDNS=yes
IPV6_PEERROUTES=yes
IPV6_FAILURE_FATAL=no
NAME=[不改]
UUID=[不改]
ONBOOT=yes
GATEWAY0=[网关]
DNS1=[DNS地址]
PROXY_METHOD=none
BROWSER_ONLY=no
PREFIX=24
GATEWAY=[网关]
ZONE=public
```

# SSH相关
## 修改端口
修改`/etc/ssh/sshd_config`，取消`port`前面的注释并修改为你想要的端口，然后执行：
```shell
systemctl restart sshd
```
**注意：务必先关闭 selinux ，并打开防火墙！**
## 修改密钥
将密钥生成在用户目录下`/.ssh/`中
```shell
ssh-keygen -t rsa 
```
## 免密登录
将本地的密钥复制到被控端的授权目录下：
```shell
ssh-copy-id -i .ssh/id_rsa.pub [username]@[IP]
```
# 防火墙
**注意：此处使用 firewall-cmd **
```shell
# 开放 tcp 协议80端口
firewall-cmd --zone=public --add-port=80/tcp --permanent
# 关闭 tcp 协议80端口
firewall-cmd --zone=public --remove-port=80/tcp --permanent
# 查看开放的端口列表
firewall-cmd --list-port
# 对防火墙操作后重启生效
firewall-cmd --reload
```
# 软件包管理（yum）相关
## 改用阿里源
```shell
# 先用原有的源安装wget！不然你会后悔的
yum install -y wget

# 备份原来的源信息文件（不然你也会后悔的）
mv /etc/yum.repos.d/CentOS-Base.repo /etc/yum.repos.d/CentOS-Base.repo.backup

# 下载阿里源文件
wget -O /etc/yum.repos.d/CentOS-Base.repo http://mirrors.aliyun.com/repo/Centos-7.repo

# 生成yum缓存（基操基操）
yum clean all
yum makecache

# 可以更新一波
yum update
```
## 添加EPEL源
用来解决很多包找不到的问题
```shell
yum -y install epel-release
```
## 安装一波流
都搞服务器运维了，这就不用写了吧……
写个说了才能想到的：
**ntp**
用来做时间同步的，方便标准化服务器时间
```shell
yum install -y ntp
```
设置服务器时间自动同步：
```shell
timedatectl set-ntp true
```
