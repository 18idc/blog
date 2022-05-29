title: ' CentOS7/8 安装 MySQL 5.7/8.0 二进制包/yum/源码/docker 等多钟安装方式'
author: q18idc.com
categories: 笔记
date: 2022-05-29 16:24:59
tags: [MySQL,CentOS]
copyright: ture
sitemap: true
---
# CentOS7/8 MySQL 5.7/8.0 安装

文档中系统镜像下载地址
CentOS 7 
<!-- more -->
系统版本：CentOS Linux release 7.9.2009 (Core)

| 镜像 | 地址                                                         |
| ---- | ------------------------------------------------------------ |
| 官网 | https://cloud.centos.org/centos/7/images/CentOS-7-x86_64-GenericCloud-2111.qcow2 |
| 腾讯 | https://mirrors.cloud.tencent.com/centos-cloud/centos/7/images/CentOS-7-x86_64-GenericCloud-2009.qcow2 |



CentOS Stream 8

| 镜像 | 地址                                                         |
| ---- | :----------------------------------------------------------- |
| 官网 | https://cloud.centos.org/centos/8-stream/x86_64/images/CentOS-Stream-GenericCloud-8-20220125.1.x86_64.qcow2 |
| 腾讯 | https://mirrors.cloud.tencent.com/centos-cloud/centos/8-stream/x86_64/images/CentOS-Stream-GenericCloud-8-20210210.0.x86_64.qcow2 |


## MySQL 5.7 安装


### CentOS 7


#### 使用二进制包安装

安装路径根据实际情况修改

数据存放目录：/data/mysql/

安装路径：/usr/local/mysql/

日志路径：/usr/local/mysql/logs/

配置文件路径：/etc/my.cnf

```bash
# 禁用SELinux
cat > /etc/selinux/config <<-'EOF'
SELINUX=disabled
SELINUXTYPE=targeted
EOF

# 先查找系统原本的数据库 mariadb
rpm -qa | grep mariadb

# 拷贝找到的列表，一个个卸载，如
rpm -e --nodeps mariadb-libs-5.5.35-3.el7.x86_64
# 或者批量卸载 
rpm -qa | grep mariadb | xargs rpm -e --nodeps 

# 安装所需包
yum install -y libaio* libncurses* libtirpc-devel wget 

# 创建mysql用户组
groupadd mysql

# 创建mysql用户
useradd -s /usr/sbin/nologin -M -g mysql mysql

# 下载包
wget https://mirrors.aliyun.com/mysql/MySQL-5.7/mysql-5.7.38-linux-glibc2.12-x86_64.tar

# 解压
tar -xvf mysql-5.7.38-linux-glibc2.12-x86_64.tar
tar -zxvf mysql-5.7.38-linux-glibc2.12-x86_64.tar.gz

# 重命名
mv mysql-5.7.38-linux-glibc2.12-x86_64/ mysql/

# 移动到 /usr/local/ 目录下
mv mysql/ /usr/local/

# 创建一些目录和文件
mkdir -p /data/mysql/
mkdir -p /usr/local/mysql/logs/ 
touch /usr/local/mysql/mysqld.pid
touch /usr/local/mysql/logs/mysqld.log

# 创建配置文件
cat >> /etc/my.cnf <<-'EOF'
[client]
default-character-set=utf8mb4

[mysql]
default-character-set=utf8mb4

[mysqld]
skip-name-resolve
basedir=/usr/local/mysql
datadir=/data/mysql
port = 3306
socket=/tmp/mysql.sock

log-error=/usr/local/mysql/logs/mysqld.log
pid-file=/usr/local/mysql/mysqld.pid

server_id=1
max_allowed_packet=256M
binlog-row-event-max-size=256m
lower_case_table_names=1
sql_mode='STRICT_TRANS_TABLES,NO_ZERO_IN_DATE,NO_ZERO_DATE,ERROR_FOR_DIVISION_BY_ZERO,NO_AUTO_CREATE_USER,NO_ENGINE_SUBSTITUTION'
character_set_server=utf8mb4
collation_server=utf8mb4_unicode_ci
log-bin=mysql-bin
max_binlog_size=1G
binlog_format=ROW
binlog_row_image=full
default-time-zone='+8:00'
max_connections=1000
EOF

# 修改目录所属用户和用户组
chown -R mysql.mysql /usr/local/mysql/
chown -R mysql.mysql /data/mysql/

# 将MySQL bin目录添加到环境变量 PATH 中 
cat >> /etc/profile <<-'EOF'
MYSQL_HOME=/usr/local/mysql
export PATH=$MYSQL_HOME/bin:$PATH
EOF
source /etc/profile

# 初始化
mysqld --defaults-file=/etc/my.cnf --initialize --user=mysql --basedir=/usr/local/mysql --datadir=/data/mysql 

# 查看初始化后的密码
cat /usr/local/mysql/logs/mysqld.log | grep password

# 启动服务 用于修改初始密码和测试
mysqld --defaults-file=/etc/my.cnf --user=mysql

# 使用初始化密码登录并修改密码 这里默认先修改为 root
mysql --defaults-file=/etc/my.cnf -uroot -p

# 修改密码为 root
alter user 'root'@'localhost' identified by 'root';
flush privileges;

# 设置为可远程连接
use mysql;
update user set host='%' where user='root';
flush privileges;
exit;

# 关闭MySQL 服务 输入上一步设置的root密码
mysqladmin -u root -p shutdown

# 创建服务文件
cat >> /lib/systemd/system/mysql.service <<-'EOF'
[Unit]
Description=MySQL
After=network.target syslog.target

[Service]
Type=simple

User=mysql
Group=mysql

TimeoutSec=0
PermissionsStartOnly=true

PIDFile=/usr/local/mysql/mysqld.pid
ExecStart=/usr/local/mysql/bin/mysqld --defaults-file=/etc/my.cnf --user=mysql
Restart=always
RestartSec=5s
PrivateTmp=false
LimitNOFILE=65536

[Install]
WantedBy=multi-user.target
EOF

# 设置权限
chmod 644 /usr/lib/systemd/system/mysql.service

# 重载服务配置
systemctl daemon-reload

# 启动 mysql 服务
systemctl start mysql

# 开启开机自启
systemctl enable mysql 

```


#### 源码方式编译安装

```bash
# 禁用SELinux
cat > /etc/selinux/config <<-'EOF'
SELINUX=disabled
SELINUXTYPE=targeted
EOF

# 先查找系统原本的数据库 mariadb
rpm -qa | grep mariadb

# 拷贝找到的列表，一个个卸载，如
rpm -e --nodeps mariadb-libs-5.5.35-3.el7.x86_64
# 或者批量卸载 
rpm -qa | grep mariadb | xargs rpm -e --nodeps 

# 创建一些目录和文件
mkdir -p /data/mysql/
mkdir -p /usr/local/mysql/logs/ 
touch /usr/local/mysql/mysqld.pid
touch /usr/local/mysql/logs/mysqld.log

# 创建用户组
groupadd mysql
# 创建用户
useradd -s /usr/sbin/nologin -M -g mysql mysql
# 安装依赖
yum install -y wget vim make cmake gcc gcc-c++ openssl openssl-devel ncurses ncurses-devel libaio* libncurses* libtirpc-devel

# 安装rpcsvc
cd /root/
wget https://github.com/thkukuk/rpcsvc-proto/releases/download/v1.4.3/rpcsvc-proto-1.4.3.tar.xz
xz -d rpcsvc-proto-1.4.3.tar.xz
tar -xvf rpcsvc-proto-1.4.3.tar
cd rpcsvc-proto-1.4.3/
./configure
make && make install

# 下载源码包
cd /root/
wget https://mirrors.aliyun.com/mysql/MySQL-5.7/mysql-boost-5.7.38.tar.gz
# 解压
tar -zxvf mysql-boost-5.7.38.tar.gz
# 进入目录
cd mysql-5.7.38

# 准备编译
# 参数参考 https://dev.mysql.com/doc/refman/5.7/en/source-configuration-options.html
mkdir build && cd build
cmake .. -DCMAKE_BUILD_TYPE=RelWithDebInfo \
-DCMAKE_INSTALL_PREFIX=/usr/local/mysql \
-DBUILD_CONFIG=mysql_release \
-DMYSQL_UNIX_ADDR=/tmp/mysql.sock \
-DMYSQL_DATADIR=/data/mysql \
-DWITH_BOOST=../boost \
-DSYSCONFDIR=/etc \
-DMYSQL_TCP_PORT=3306 \
-DWITH_ARCHIVE_STORAGE_ENGINE=1 \
-DWITH_FEDERATED_STORAGE_ENGINE=1 \
-DWITH_INNOBASE_STORAGE_ENGINE=1 \
-DWITH_PARTITION_STORAGE_ENGINE=1 \
-DWITH_BLACKHOLE_STORAGE_ENGINE=1 \
-DWITH_MYISAM_STORAGE_ENGINE=1 \
-DWITH_EXTRA_CHARSETS=all \
-DDEFAULT_CHARSET=utf8mb4 \
-DDEFAULT_COLLATION=utf8mb4_general_ci \
-DWITH_EMBEDDED_SERVER=1 \
-DENABLED_LOCAL_INFILE=1 \
-DWITH_DEBUG=0 \
-DWITH_SSL=system

# 编译安装
make -j4 && make -j4 install

# 后续步骤参考 二进制包安装 中 创建配置文件 及后续步骤

```


#### 卸载二进制包安装与源码编译安装

```bash
### 备份数据目录和配置文件  /data/mysql/ /etc/my.cnf

systemctl disable mysql
systemctl stop mysql
rm -rf /etc/my.cnf
rm -rf /data/mysql/
rm -rf /usr/local/mysql/
rm -rf /usr/lib/systemd/system/mysql.service
systemctl daemon-reload
```

#### yum 安装与卸载

官方文档：https://dev.mysql.com/doc/refman/5.7/en/linux-installation-yum-repo.html

https://dev.mysql.com/doc/refman/8.0/en/linux-installation-yum-repo.html

默认数据存放目录：/var/lib/mysql/

默认配置文件路径：/etc/my.cnf

```bash
# CentOS8系统需要禁用自带的mysql模块 CentOS8系统执行
# yum module disable -y mysql

# 先查找系统原本的数据库 mariadb
rpm -qa | grep mariadb

# 拷贝找到的列表，一个个卸载，如
#rpm -e --nodeps mariadb-libs-5.5.35-3.el7.x86_64
# 或者批量卸载 
rpm -qa | grep mariadb | xargs rpm -e --nodeps 

# 下载yum源
wget https://repo.mysql.com/mysql57-community-release-el7.rpm

# 安装MySQL5.7 yum源
rpm -ivh mysql57-community-release-el7.rpm

# 导入GPG Key
rpm --import https://repo.mysql.com/RPM-GPG-KEY-mysql-2022
rpm --import https://repo.mysql.com/RPM-GPG-KEY-mysql

# 安装MySQL5.7
yum install -y mysql-community-server

# 启动
systemctl start mysqld

# 停止
systemctl stop mysqld

# 查看启动状态
systemctl status mysqld

# 设置开机自启
systemctl enable mysqld

# 查看生成的root密码
cat /var/log/mysqld.log | grep password


# 查找安装的mysql
rpm -qa | grep mysql
# 停止并卸载
systemctl stop mysqld
systemctl disable mysqld
rpm -qa | grep mysql | xargs rpm -e --nodeps 

```

### CentOS Stream 8

参考CentOS 7系统下安装即可


### Docker

开发测试环境可以推荐使用此方法安装，生产环境不推荐使用此方法安装

```bash
echo 'Asia/Shanghai' > /etc/timezone
# 1.创建mysql的配置文件
mkdir -p /srv/mysql/conf /srv/mysql/logs /srv/mysql/data

# 2.创建mysql配置/srv/mysql/conf/custom.cnf
cat >> /srv/mysql/conf/custom.cnf <<-'EOF'
[mysqld]
skip-name-resolve
server_id=1
max_allowed_packet=256M
binlog-row-event-max-size=256m
lower_case_table_names=1
sql_mode='STRICT_TRANS_TABLES,NO_ZERO_IN_DATE,NO_ZERO_DATE,ERROR_FOR_DIVISION_BY_ZERO,NO_AUTO_CREATE_USER,NO_ENGINE_SUBSTITUTION'
character_set_server=utf8mb4
collation_server=utf8mb4_unicode_ci
log-bin=mysql-bin
max_binlog_size=1G
binlog_format=ROW
binlog_row_image=full
default-time-zone='+8:00'
max_connections=512
EOF

# 3.运行mysql5.7实例
docker run --net=host -d --name mysql --privileged --restart=always -v /srv/mysql/conf:/etc/mysql/conf.d -v /srv/mysql/logs:/var/log/mysql -v /srv/mysql/data:/var/lib/mysql -v /etc/timezone:/etc/timezone -e TZ=Asia/Shanghai -e MYSQL_ROOT_PASSWORD=root mysql:5.7

```


## MySQL 8.0 安装


### CentOS 7 


#### 使用二进制包安装

```bash
# 禁用SELinux
cat > /etc/selinux/config <<-'EOF'
SELINUX=disabled
SELINUXTYPE=targeted
EOF

# 先查找系统原本的数据库 mariadb
rpm -qa | grep mariadb

# 拷贝找到的列表，一个个卸载，如
rpm -e --nodeps mariadb-libs-5.5.35-3.el7.x86_64
# 或者批量卸载 
rpm -qa | grep mariadb | xargs rpm -e --nodeps 

# 安装所需包
yum install -y libaio* libncurses* libtirpc-devel wget

# 创建mysql用户组
groupadd mysql

# 创建mysql用户
useradd -s /usr/sbin/nologin -M -g mysql mysql

# 下载包
wget https://mirrors.aliyun.com/mysql/MySQL-8.0/mysql-8.0.29-linux-glibc2.12-x86_64.tar

# 解压
tar -xvf mysql-8.0.29-linux-glibc2.12-x86_64.tar
tar -xJvf mysql-8.0.29-linux-glibc2.12-x86_64.tar.xz

# 重命名
mv mysql-8.0.29-linux-glibc2.12-x86_64/ mysql/

# 移动到 /usr/local/ 目录下
mv mysql/ /usr/local/

# 创建一些目录和文件
mkdir -p /data/mysql/
mkdir -p /usr/local/mysql/logs/ 
touch /usr/local/mysql/mysqld.pid
touch /usr/local/mysql/logs/mysqld.log

# 创建配置文件
cat >> /etc/my.cnf <<-'EOF'
[client]
default-character-set=utf8mb4

[mysql]
default-character-set=utf8mb4

[mysqld]
skip-name-resolve
basedir=/usr/local/mysql
datadir=/data/mysql
port=3306
mysqlx_port=33060
socket=/tmp/mysql.sock

log-error=/usr/local/mysql/logs/mysqld.log
pid-file=/usr/local/mysql/mysqld.pid

server_id=1
max_allowed_packet=256M
binlog-row-event-max-size=256m
lower_case_table_names=1
sql_mode='STRICT_TRANS_TABLES,NO_ZERO_IN_DATE,NO_ZERO_DATE,ERROR_FOR_DIVISION_BY_ZERO,NO_ENGINE_SUBSTITUTION'
character_set_server=utf8mb4
collation_server=utf8mb4_unicode_ci
log-bin=mysql-bin
max_binlog_size=1G
binlog_format=ROW
binlog_row_image=full
default-time-zone='+8:00'
max_connections=1000
EOF

# 修改目录所属用户和用户组
chown -R mysql.mysql /usr/local/mysql/
chown -R mysql.mysql /data/mysql/

# 将MySQL bin目录添加到环境变量 PATH 中 
cat >> /etc/profile <<-'EOF'
MYSQL_HOME=/usr/local/mysql
export PATH=$MYSQL_HOME/bin:$PATH
EOF
source /etc/profile

# 初始化
mysqld --defaults-file=/etc/my.cnf --initialize --user=mysql --basedir=/usr/local/mysql --datadir=/data/mysql 

# 查看初始化后的密码
cat /usr/local/mysql/logs/mysqld.log | grep password

# 启动服务 用于修改初始密码和测试
mysqld --defaults-file=/etc/my.cnf --user=mysql

# 使用初始化密码登录并修改密码 这里默认先修改为 root
mysql --defaults-file=/etc/my.cnf -uroot -p

# 修改密码为 root
alter user 'root'@'localhost' identified by 'root';
flush privileges;

# 设置为可远程连接
use mysql;
update user set host='%' where user='root';
flush privileges;
exit;

# 关闭MySQL 服务 输入上一步设置的root密码
mysqladmin -u root -p shutdown

# 创建服务文件
cat >> /lib/systemd/system/mysql.service <<-'EOF'
[Unit]
Description=MySQL
After=network.target syslog.target

[Service]
Type=simple

User=mysql
Group=mysql

TimeoutSec=0
PermissionsStartOnly=true

PIDFile=/usr/local/mysql/mysqld.pid
ExecStart=/usr/local/mysql/bin/mysqld --defaults-file=/etc/my.cnf --user=mysql
Restart=always
RestartSec=5s
PrivateTmp=false
LimitNOFILE=65536

[Install]
WantedBy=multi-user.target
EOF

# 设置权限
chmod 644 /usr/lib/systemd/system/mysql.service

# 重载服务配置
systemctl daemon-reload

# 启动 mysql 服务
systemctl start mysql

# 开启开机自启
systemctl enable mysql 

```


#### 源码方式编译安装

```bash
# 禁用SELinux
cat > /etc/selinux/config <<-'EOF'
SELINUX=disabled
SELINUXTYPE=targeted
EOF

# 先查找系统原本的数据库 mariadb
rpm -qa | grep mariadb

# 拷贝找到的列表，一个个卸载，如
rpm -e --nodeps mariadb-libs-5.5.68-1.el7.x86_64
# 或者批量卸载 
rpm -qa | grep mariadb | xargs rpm -e --nodeps 

# 创建一些目录和文件
mkdir -p /data/mysql/
mkdir -p /usr/local/mysql/logs/ 
touch /usr/local/mysql/mysqld.pid
touch /usr/local/mysql/logs/mysqld.log

# 创建用户组
groupadd mysql
# 创建用户
useradd -s /usr/sbin/nologin -M -g mysql mysql
# 安装依赖
yum install -y epel-release
yum install -y wget vim make gcc gcc-c++ openssl openssl-devel openssl-libs ncurses ncurses-devel libaio* libncurses* libtirpc-devel bison m4 bzip2 libudev-devel epel-release
yum install -y cmake3

# GCC版本过低 在CentOS7下编译MySQL8需要GCC7.1+版本
#查看gcc版本，CentOS7默认安装4.8.5
gcc -v
#安装 devtoolset
yum -y install centos-release-scl
yum -y install devtoolset-11-gcc devtoolset-11-gcc-c++ devtoolset-11-binutils
#scl命令启用只是临时的，退出shell或重启就会恢复原系统gcc版本。
scl enable devtoolset-11 bash
#查看gcc版本
gcc -v

# 下载MySQL8源码包
cd /root/
wget https://mirrors.aliyun.com/mysql/MySQL-8.0/mysql-boost-8.0.29.tar.gz
# 解压
tar -zxvf mysql-boost-8.0.29.tar.gz
# 进入目录
cd mysql-8.0.29

# 准备编译
# 参数参考 https://dev.mysql.com/doc/refman/8.0/en/source-configuration-options.html
mkdir build && cd build
cmake3 .. -DCMAKE_BUILD_TYPE=RelWithDebInfo \
-DCMAKE_INSTALL_PREFIX=/usr/local/mysql \
-DBUILD_CONFIG=mysql_release \
-DMYSQL_UNIX_ADDR=/tmp/mysql.sock \
-DMYSQL_DATADIR=/data/mysql \
-DWITH_BOOST=../boost \
-DSYSCONFDIR=/etc \
-DMYSQL_TCP_PORT=3306 \
-DWITH_ARCHIVE_STORAGE_ENGINE=1 \
-DWITH_FEDERATED_STORAGE_ENGINE=1 \
-DWITH_INNOBASE_STORAGE_ENGINE=1 \
-DWITH_BLACKHOLE_STORAGE_ENGINE=1 \
-DWITH_MYISAM_STORAGE_ENGINE=1 \
-DDEFAULT_CHARSET=utf8mb4 \
-DDEFAULT_COLLATION=utf8mb4_general_ci \
-DENABLED_LOCAL_INFILE=1 \
-DWITH_DEBUG=0 \
-DWITH_SSL=system \
-DFORCE_INSOURCE_BUILD=1

# 编译安装
make -j4 && make -j4 install

# 后续步骤参考 二进制包安装 中 创建配置文件 及后续步骤

```


#### yum源安装与卸载

官方文档：https://dev.mysql.com/doc/refman/8.0/en/linux-installation-yum-repo.html

默认数据存放目录：/var/lib/mysql/

默认配置文件路径：/etc/my.cnf

```bash
# 先查找系统原本的数据库 mariadb
rpm -qa | grep mariadb

# 拷贝找到的列表，一个个卸载，如
#rpm -e --nodeps mariadb-libs-5.5.35-3.el7.x86_64
# 或者批量卸载 
rpm -qa | grep mariadb | xargs rpm -e --nodeps 

# 下载MySQL8 yum源
wget https://repo.mysql.com/mysql80-community-release-el7.rpm
# 安装MySQL8 yum源
yum localinstall -y mysql80-community-release-el7.rpm

# 导入GPG Key
rpm --import https://repo.mysql.com/RPM-GPG-KEY-mysql-2022
rpm --import https://repo.mysql.com/RPM-GPG-KEY-mysql

# 安装MySQL8
yum install -y mysql-community-server

# 启动
systemctl start mysqld

# 停止
systemctl stop mysqld

# 查看启动状态
systemctl status mysqld

# 设置开机自启
systemctl enable mysqld

# 查看生成的root密码
cat /var/log/mysqld.log | grep password


# 查找安装的mysql
rpm -qa | grep mysql
# 停止并卸载
systemctl stop mysqld
systemctl disable mysqld
rpm -qa | grep mysql | xargs rpm -e --nodeps 

```



### CentOS Stream 8


#### 使用二进制包安装

```bash
# 禁用SELinux
cat > /etc/selinux/config <<-'EOF'
SELINUX=disabled
SELINUXTYPE=targeted
EOF

# 先查找系统原本的数据库 mariadb
rpm -qa | grep mariadb

# 拷贝找到的列表，一个个卸载，如
rpm -e --nodeps mariadb-libs-5.5.35-3.el7.x86_64
# 或者批量卸载 
rpm -qa | grep mariadb | xargs rpm -e --nodeps 

# 安装所需包
yum install -y libaio* libncurses* libtirpc* ncurses-compat-libs wget vim 

# 创建mysql用户组
groupadd mysql

# 创建mysql用户
useradd -s /usr/sbin/nologin -M -g mysql mysql

# 下载包
wget https://mirrors.aliyun.com/mysql/MySQL-8.0/mysql-8.0.29-linux-glibc2.12-x86_64.tar

# 解压
tar -xvf mysql-8.0.29-linux-glibc2.12-x86_64.tar
tar -xJvf mysql-8.0.29-linux-glibc2.12-x86_64.tar.xz

# 重命名
mv mysql-8.0.29-linux-glibc2.12-x86_64/ mysql/

# 移动到 /usr/local/ 目录下
mv mysql/ /usr/local/

# 创建一些目录和文件
mkdir -p /data/mysql/
mkdir -p /usr/local/mysql/logs/ 
touch /usr/local/mysql/mysqld.pid
touch /usr/local/mysql/logs/mysqld.log

# 创建配置文件
cat > /etc/my.cnf <<-'EOF'
[client]
default-character-set=utf8mb4

[mysql]
default-character-set=utf8mb4

[mysqld]
skip-name-resolve
basedir=/usr/local/mysql
datadir=/data/mysql
port=3306
mysqlx_port=33060
socket=/tmp/mysql.sock

log-error=/usr/local/mysql/logs/mysqld.log
pid-file=/usr/local/mysql/mysqld.pid

server_id=1
max_allowed_packet=256M
binlog-row-event-max-size=256m
lower_case_table_names=1
sql_mode='STRICT_TRANS_TABLES,NO_ZERO_IN_DATE,NO_ZERO_DATE,ERROR_FOR_DIVISION_BY_ZERO,NO_ENGINE_SUBSTITUTION'
character_set_server=utf8mb4
collation_server=utf8mb4_unicode_ci
log-bin=mysql-bin
max_binlog_size=1G
binlog_format=ROW
binlog_row_image=full
default-time-zone='+8:00'
max_connections=1000
EOF

# 修改目录所属用户和用户组
chown -R mysql.mysql /usr/local/mysql/
chown -R mysql.mysql /data/mysql/

# 将MySQL bin目录添加到环境变量 PATH 中 
cat >> /etc/profile <<-'EOF'
MYSQL_HOME=/usr/local/mysql
export PATH=$MYSQL_HOME/bin:$PATH
EOF
source /etc/profile

# 初始化
mysqld --defaults-file=/etc/my.cnf --initialize --user=mysql --basedir=/usr/local/mysql --datadir=/data/mysql 

# 查看初始化后的密码
cat /usr/local/mysql/logs/mysqld.log | grep password

# 启动服务 用于修改初始密码和测试
mysqld --defaults-file=/etc/my.cnf --user=mysql

# 使用初始化密码登录并修改密码 这里默认先修改为 root
mysql --defaults-file=/etc/my.cnf -uroot -p

# 修改密码为 root
alter user 'root'@'localhost' identified by 'root';
flush privileges;

# 设置为可远程连接
use mysql;
update user set host='%' where user='root';
flush privileges;
exit;

# 关闭MySQL 服务 输入上一步设置的root密码
mysqladmin -u root -p shutdown

# 创建服务文件
cat >> /lib/systemd/system/mysql.service <<-'EOF'
[Unit]
Description=MySQL
After=network.target syslog.target

[Service]
Type=simple

User=mysql
Group=mysql

TimeoutSec=0
PermissionsStartOnly=true

PIDFile=/usr/local/mysql/mysqld.pid
ExecStart=/usr/local/mysql/bin/mysqld --defaults-file=/etc/my.cnf --user=mysql
Restart=always
RestartSec=5s
PrivateTmp=false
LimitNOFILE=65536

[Install]
WantedBy=multi-user.target
EOF

# 设置权限
chmod 644 /usr/lib/systemd/system/mysql.service

# 重载服务配置
systemctl daemon-reload

# 启动 mysql 服务
systemctl start mysql

# 开启开机自启
systemctl enable mysql 

```


#### 源码方式编译安装

```bash
# 禁用SELinux
cat > /etc/selinux/config <<-'EOF'
SELINUX=disabled
SELINUXTYPE=targeted
EOF

# 先查找系统原本的数据库 mariadb
rpm -qa | grep mariadb

# 拷贝找到的列表，一个个卸载，如
rpm -e --nodeps mariadb-libs-5.5.68-1.el7.x86_64
# 或者批量卸载 
rpm -qa | grep mariadb | xargs rpm -e --nodeps 

# 创建一些目录和文件
mkdir -p /data/mysql/
mkdir -p /usr/local/mysql/logs/ 
touch /usr/local/mysql/mysqld.pid
touch /usr/local/mysql/logs/mysqld.log

# 创建用户组
groupadd mysql
# 创建用户
useradd -s /usr/sbin/nologin -M -g mysql mysql
# 安装依赖
yum install -y wget vim make cmake gcc gcc-c++ openssl* ncurses* libaio* libncurses* libtirpc-devel bison m4 bzip2 libudev-devel gcc-toolset-11-gcc gcc-toolset-11-gcc-c++ gcc-toolset-11-binutils

# 安装rpcsvc
cd /root/
wget https://github.com/thkukuk/rpcsvc-proto/releases/download/v1.4.3/rpcsvc-proto-1.4.3.tar.xz
xz -d rpcsvc-proto-1.4.3.tar.xz
tar -xvf rpcsvc-proto-1.4.3.tar
cd rpcsvc-proto-1.4.3/
./configure
make && make install

# 下载MySQL8源码包
cd /root/
wget https://mirrors.aliyun.com/mysql/MySQL-8.0/mysql-boost-8.0.29.tar.gz
# 解压
tar -zxvf mysql-boost-8.0.29.tar.gz
# 进入目录
cd mysql-8.0.29

# 准备编译
# 参数参考 https://dev.mysql.com/doc/refman/8.0/en/source-configuration-options.html
mkdir build && cd build
cmake3 .. -DCMAKE_BUILD_TYPE=RelWithDebInfo \
-DCMAKE_INSTALL_PREFIX=/usr/local/mysql \
-DBUILD_CONFIG=mysql_release \
-DMYSQL_UNIX_ADDR=/tmp/mysql.sock \
-DMYSQL_DATADIR=/data/mysql \
-DWITH_BOOST=../boost \
-DSYSCONFDIR=/etc \
-DMYSQL_TCP_PORT=3306 \
-DWITH_ARCHIVE_STORAGE_ENGINE=1 \
-DWITH_FEDERATED_STORAGE_ENGINE=1 \
-DWITH_INNOBASE_STORAGE_ENGINE=1 \
-DWITH_BLACKHOLE_STORAGE_ENGINE=1 \
-DWITH_MYISAM_STORAGE_ENGINE=1 \
-DDEFAULT_CHARSET=utf8mb4 \
-DDEFAULT_COLLATION=utf8mb4_general_ci \
-DENABLED_LOCAL_INFILE=1 \
-DWITH_DEBUG=0 \
-DWITH_SSL=system \
-DFORCE_INSOURCE_BUILD=1

# 编译安装
make -j4 && make -j4 install

# 后续步骤参考 二进制包安装 中 创建配置文件 及后续步骤

```


#### yum源安装与卸载

官方文档：https://dev.mysql.com/doc/refman/8.0/en/linux-installation-yum-repo.html

默认数据存放目录：/var/lib/mysql/

默认配置文件路径：/etc/my.cnf

```bash
# 禁用系统默认的mysql模块
yum module disable mysql
# 下载MySQL8 yum源
wget https://repo.mysql.com/mysql80-community-release-el8.rpm
# 安装MySQL8 yum源
yum install -y mysql80-community-release-el8.rpm
# 安装MySQL8
yum install -y mysql-community-server

# 启动
systemctl start mysqld

# 停止
systemctl stop mysqld

# 查看启动状态
systemctl status mysqld

# 设置开机自启
systemctl enable mysqld

# 查看生成的root密码
cat /var/log/mysqld.log | grep password


# 查找安装的mysql
rpm -qa | grep mysql
# 停止并卸载
systemctl stop mysqld
systemctl disable mysqld
rpm -qa | grep mysql | xargs rpm -e --nodeps 

```


### Docker

```bash
echo 'Asia/Shanghai' > /etc/timezone
# 1.创建mysql的配置文件
mkdir -p /srv/mysql/conf /srv/mysql/logs /srv/mysql/data

# 2.创建mysql配置/srv/mysql/conf/custom.cnf
cat >> /srv/mysql/conf/custom.cnf <<-'EOF'
[mysqld]
port=3306
mysqlx_port=33060
skip-name-resolve
server_id=1
max_allowed_packet=256M
binlog-row-event-max-size=256m
lower_case_table_names=1
sql_mode='STRICT_TRANS_TABLES,NO_ZERO_IN_DATE,NO_ZERO_DATE,ERROR_FOR_DIVISION_BY_ZERO,NO_ENGINE_SUBSTITUTION'
character_set_server=utf8mb4
collation_server=utf8mb4_unicode_ci
log-bin=mysql-bin
max_binlog_size=1G
binlog_format=ROW
binlog_row_image=full
default-time-zone='+8:00'
max_connections=512
EOF

# 3.运行mysql8.0实例
docker run --net=host -d --name mysql8 --privileged --restart=always -v /srv/mysql/conf:/etc/mysql/conf.d -v /srv/mysql/logs:/var/log/mysql -v /srv/mysql/data:/var/lib/mysql -v /etc/timezone:/etc/timezone -e TZ=Asia/Shanghai -e MYSQL_ROOT_PASSWORD=root mysql:8.0

```
