---
title: CentOS7安装Redis6.0.6
tags: CentOS
categories: 笔记
date: 2020-08-01 09:51:40
copyright: ture
---
# 准备
[Redis官网](https://redis.io/)
[CentOS官网](https://www.centos.org/)
[redis-6.0.6.tar.gz](http://download.redis.io/releases/redis-6.0.6.tar.gz)
[CentOS-7-x86_64-Everything-2003.iso](https://mirrors.aliyun.com/centos/7.8.2003/isos/x86_64/CentOS-7-x86_64-Everything-2003.iso)

# 安装Redis

<!-- more -->

```bash
#安装必要依赖包
yum install -y gcc wget tar vim

#下载Redis6.0.6源码包
wget http://download.redis.io/releases/redis-6.0.6.tar.gz

#解压Redis源码包
tar -zxvf redis-6.0.6.tar.gz

#进入解压后的目录
cd redis-6.0.6

#指定Redis安装目录 编译并安装
make PREFIX=/usr/local/redis install 

#创建配置文件目录
mkdir /usr/local/redis/conf/

#复制配置文件到Redis安装目录中
cp redis.conf /usr/local/redis/conf/

#编辑环境变量文件
vim /etc/profile

#添加下面两行内容
export REDIS=/usr/local/redis
export PATH=:$PATH:$REDIS/bin

#退出并保存 按键盘上的 ESC 按键 输入 :wq
ESC :wq

#让环境变量文件生效
source /etc/profile

#启动Redis
redis-server /usr/local/redis/conf/redis.conf
```

# 报错
## 报错1：server.c:xxxx:xx: error: ‘xxxxxxxx’ has no member named ‘xxxxx’
```bash
server.c:5159:49: error: struct redisServer has no member named supervised
     int background = server.daemonize && !server.supervised;
                                                 ^
server.c:5163:29: error: struct redisServer has no member named pidfile
     if (background || server.pidfile) createPidFile();
```
解决办法 
```bash
#查看gcc版本是否在5.3以上，CentOS7默认安装4.8.5
gcc -v

#升级gcc
yum -y install centos-release-scl
yum -y install devtoolset-9-gcc devtoolset-9-gcc-c++ devtoolset-9-binutils

#scl命令启用只是临时的，退出shell或重启就会恢复原系统gcc版本。
scl enable devtoolset-9 bash

#查看gcc版本是否已被升级
gcc -v

#重新执行编译安装命令
make PREFIX=/usr/local/redis install 
```
# 启动警告
## WARNING overcommit_memory is set to 0!
```bash
#编辑配置文件
vim /etc/sysctl.conf

#添加这一行
vm.overcommit_memory=1

#保存并退出 按键盘上的 ESC 键 输入 :wq
ESC :wq

#使配置生效
sysctl -p
```
## WARNING you have Transparent Huge Pages (THP) support enabled in your kernel.
```bash
echo never > /sys/kernel/mm/transparent_hugepage/enabled

#编辑启动脚本
vim /etc/rc.local

if test -f /sys/kernel/mm/transparent_hugepage/enabled; then
echo never > /sys/kernel/mm/transparent_hugepage/enabled
fi

#保存并退出 按键盘上的 ESC 键 输入 :wq
ESC :wq
```

## WARNING: The TCP backlog setting of 511 cannot be enforced because /proc/sys/net/core/somaxconn
```bash
#编辑配置文件
vim /etc/sysctl.conf

#添加这一行
net.core.somaxconn=1024

#保存并退出 按键盘上的 ESC 键 输入 :wq
ESC :wq

#使配置生效
sysctl -p
```
