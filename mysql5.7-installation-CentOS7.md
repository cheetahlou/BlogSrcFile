---
title: mysql5.7-installation-CentOS7
date: 2017-12-08 10:37:59
tags:
- linux
- MySQL
---





CentOS7环境下安装MySQL 5.7记录Remarks一哈。

<!--more-->

### 核心思想——东拼西凑

 - 卸载
> - 查看已安装server
 rpm -qa | grep -i mysql  

>>mysql-community-common-5.7.20-1.el7.x86_64
>>mysql57-community-release-el7-8.noarch
>>mysql-community-libs-5.7.20-1.el7.x86_64
>>mysql-community-server-5.7.20-1.el7.x86_64
>>mysql-community-client-5.7.20-1.el7.x86_64
>>
>
 - yum卸载
>>yum remove mysql-community-common-5.7.20-1.el7.x86_64 mysql-community-server-5.7.20-1.el7.x86_64 …（空格隔开） 

安装参考：
>[（笔记）CentOS 7 安装与卸载MySQL 5.7跳坑](http://www.jianshu.com/p/e54ff5283f18)
[CentOS 7.4使用Yum源安装MySQL 5.7.20](http://blog.csdn.net/cmzsteven/article/details/78515756)

步骤：
>1. 首先进入本机的源文件目录
cd /usr/local/src
2. 使用wget下载官方yum源的rpm包：
>
```
wget https://dev.mysql.com/get/mysql57-community-release-el7-11.noarch.rpm
```
3. 安装rpm包：
`rpm -ivh mysql57-community-release-el7-11.noarch.rpm`
4. 再次使用yum来安装mysql-server:
yum install -y mysql-server
5. 安装完成后，启动mysqld服务：
`systemctl start mysqld`
查看是否成功启动：
`ps aux|grep mysqld`
6. 设置mysqld服务开机自启动：
`systemctl enable mysqld`
7. 使用初始密码登录
由于MySQL从5.7开始不允许首次安装后，使用空密码进行登录，系统会随机生成一个密码以供管理员首次登录使用，这个密码记录在/var/log/mysqld.log文件中，使用下面的命令可以查看此密码：
`cat /var/log/mysqld.log|grep 'A temporary password'`
>>2017-11-12T13:35:37.013617Z 1 [Note] A temporary password is generated for root@localhost: bkv,dy,)o7Ss
>>
>

最后一行冒号后面的部分bkv,dy,)o7Ss就是初始密码。 
使用此密码登录MySQL:
`mysql -u root -p`
8. 更改默认密码：
切换数据库：
`use mysql;`
修改root密码：
`alter user 'root'@'localhost' identified by 'your_password';`
密码要求包含大小写字母、数字及标点符号，长度应该在6位以上。 
重新使用新的密码登录，如果可以正常登录说明你的MySQL已经成功安装在CentOS 7.4上了

 - 授权远程登录

>mysql -uroot -pxxxxx
//切换至mysql的数据库
use mysql;
select user,host from user;
+-----------+-----------+
| user | host |
+-----------+-----------+
| mysql.sys | localhost |
| root | localhost |
| user1 | localhost |
+-----------+-----------+
3 rows in set (0.00 sec)
//我们能看到root 用户 后面跟的是localhost 如果需要指定具体的ip，那么就修改为具体的ip，如果需要开通所有就修改为%
update user set host = '%' where user = 'root';
//修改完成退出mysql
+-----------+-----------+
| user | host |
+-----------+-----------+
| root | % |
| mysql.sys | localhost |
| user1 | localhost |
+-----------+-----------+
3 rows in set (0.00 sec)

 -  - 设置root用户远程登录：


```
$mysql -u root -p
mysql>use mysql;
mysql>update user set host='%' where user='root';//设置可以远程访问
mysql>grant all privileges on *.* to 'root'@'%' with grant option;
mysql>flush privileges;    //设置远程访问权限
```


 - - - 或 添加一个远程登录用户：
>mysql> GRANT ALL PRIVILEGES ON *.* TO 'admin'@'%' IDENTIFIED BY 'secret' WITH GRANT OPTION;
>
>

 - -  - 修改或添加/etc/my.cnf目录下的my.cnf文件里面
bind-address = 0.0.0.0

 - 防火墙开放3306端口
> 1.阿里云服务器安全组策略中入方向打开3306端口
>
> 2.其他（未验证）：
>>1、打开防火墙配置文件
vi  /etc/sysconfig/iptables
2、增加下面一行
-A INPUT -m state --state NEW -m tcp -p tcp --dport 3306 -j ACCEPT
3、重启防火墙
service  iptables restart
注意：增加的开放3306端口的语句一定要在icmp-host-prohibited之前

 - 配置默认编码为 utf8

MySQL 默认为 latin1, 一般修改为 UTF-8
>$ vi /etc/my.cnf
[mysqld]
 #在myslqd下添加如下键值对
character_set_server=utf8
init_connect='SET NAMES utf8'
修改前：
>>mysql> SHOW VARIABLES LIKE 'character%';
+--------------------------+----------------------------+
| Variable_name            | Value                      |
+--------------------------+----------------------------+
| character_set_client     | utf8                       |
| character_set_connection | utf8                       |
| character_set_database   | latin1                     |
| character_set_filesystem | binary                     |
| character_set_results    | utf8                       |
| character_set_server     | latin1                     |
| character_set_system     | utf8                       |
| character_sets_dir       | /usr/share/mysql/charsets/ |
+--------------------------+----------------------------+
8 rows in set (0.00 sec)
>>
>
修改后：
>>mysql> SHOW VARIABLES LIKE 'character%';
+--------------------------+----------------------------+
| Variable_name            | Value                      |
+--------------------------+----------------------------+
| character_set_client     | utf8                       |
| character_set_connection | utf8                       |
| character_set_database   | utf8                       |
| character_set_filesystem | binary                     |
| character_set_results    | utf8                       |
| character_set_server     | utf8                       |
| character_set_system     | utf8                       |
| character_sets_dir       | /usr/share/mysql/charsets/ |
+--------------------------+----------------------------+
8 rows in set (0.00 sec)



>重启mysql服务使配置生效：
`systemctl restart mysqld`

 - 默认配置文件：
 >**默认配置文件路径：  **
配置文件：/etc/my.cnf  
日志文件：/var/log/mysqld.log  
服务启动脚本：/usr/lib/systemd/system/mysqld.service  
socket文件：/var/run/mysqld/mysqld.pid



 