---
layout: post
title: CentOS安装Mysql
subtitle: centos install mysql
categories: linux
tags: linux
sidebar: []
---

注意：
一、安装MySQL
1、下载安装包 mysql-5.6.36-linux-glibc2.5-x86_64.tar.gz
下载地址https://dev.mysql.com/downloads/mysql/5.6.html
选择如下选项
https://note.youdao.com/yws/res/708/67A6B3C4522F475E8736FA69E2CD061C
Center
 
下载这个版本：
https://note.youdao.com/yws/res/707/4C077577AD824E68BEA09FD073C9F13A
Center
 
2、卸载系统自带的Mariadb
rpm -qa|grep mariadb         //查询出已安装的mariadb
rpm -e --nodeps 文件名      //卸载 ， 文件名为使用rpm -qa|grep mariadb 命令查出的所有文件
3、删除etc目录下的my.cnf文件
       rm /etc/my.cnf
4、 执行以下命令来创建mysql用户组
groupadd mysql
5、执行以下命令来创建一个用户名为mysql的用户并加入mysql用户组
useradd -g mysql mysql
6、将下载的二进制压缩包放到/usr/local/目录下。
7、解压安装包
tar -zxvfmysql-5.6.36-linux-glibc2.5-x86_64.tar.gz
8、将解压好的文件夹重命名为mysql
9、在etc下新建配置文件my.cnf，并在该文件内添加以下代码：
[mysql]
# 设置mysql客户端默认字符集
default-character-set=utf8
socket=/var/lib/mysql/mysql.sock
[mysqld]
skip-name-resolve
#设置3306端口
port=3306
socket=/var/lib/mysql/mysql.sock
# 设置mysql的安装目录
basedir=/usr/local/mysql
# 设置mysql数据库的数据的存放目录
datadir=/usr/local/mysql/data
# 允许最大连接数
max_connections=200
# 服务端使用的字符集默认为8比特编码的latin1字符集
character-set-server=utf8
# 创建新表时将使用的默认存储引擎
default-storage-engine=INNODB
lower_case_table_names=1
max_allowed_packet=16M
10、创建步骤9中用到的目录并将其用户设置为mysql
mkdir /var/lib/mysql
mkdir /var/lib/mysql/mysql
chown -R mysql:mysql /var/lib/mysql
chown -R mysql:mysql /var/lib/mysql/mysql
11、进入安装mysql软件目录
cd /usr/local/mysql
chown -R mysql:mysql ./　　                             #修改当前目录拥有者为mysql用户
./scripts/mysql_install_db --user=mysql         #安装数据库
chown -R mysql:mysql data                              #修改当前data目录拥有者为mysql用户
 
到此数据库安装完毕！
 

二、配置MySQL
1、授予my.cnf的最大权限。
chown 777 /etc/my.cnf
设置开机自启动服务控制脚本：
2、复制启动脚本到资源目录
cp ./support-files/mysql.server /etc/rc.d/init.d/mysqld
3、增加mysqld服务控制脚本执行权限
chmod +x /etc/rc.d/init.d/mysqld
4、将mysqld服务加入到系统服务
chkconfig --add mysqld
5、检查mysqld服务是否已经生效
chkconfig --list mysqld

命令输出类似下面的结果：

mysqld 0:off 1:off 2:on 3:on 4:on 5:on 6:off

表明mysqld服务已经生效，在2、3、4、5运行级别随系统启动而自动启动，以后可以使用service命令控制mysql的启动和停止。
6、启动msql（停止mysqld服务：service mysqld stop）
service mysqld start
7、将mysql的bin目录加入PATH环境变量，编辑/etc/profile文件
vi /etc/profile
在文件最后添加如下信息：
export PATH=$PATH:/usr/local/mysql/bin
执行下面的命令使所做的更改生效：
. /etc/profile
8、以root账户登陆mysql，默认是没有密码
mysql -u root -p
9、设置root账户密码 注意下面的you password改成你的要修改的密码
 use mysql
update user set password=password('you password') where user='root'and host='localhost';
10、设置远程主机登录，注意下面的your username 和 your password改成你需要设置的用户和密码
GRANT ALL PRIVILEGES ON *.* TO'root'@'%' IDENTIFIED BY 'laiganhuo' WITH GRANT OPTION;
FLUSH PRIVILEGES ;
