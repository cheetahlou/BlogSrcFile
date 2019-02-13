---
title: CentOS7-nginx-multi-tomcat
date: 2019-02-01 23:37:00
tags: linux
---



CentOS7环境下安装多个tomcat应用服务器

<!--more-->


﻿>*注：本文 nginx版本：1.12.2      Tomcat版本: 9.0.2*
###*yum安装
>`[root@izbp1di2a9vvogy93gw0onz local]# rpm -ivh https://nginx.org/packages/centos/7/noarch/RPMS/nginx-release-centos-7-0.el7.ngx.noarch.rpm`
>`[root@izbp1di2a9vvogy93gw0onz local]# yum install nginx`

###*启动
>`[root@izbp1di2a9vvogy93gw0onz local]# systemctl start nginx.service`
**启动报错：**
>><font color="red">Job for nginx.service failed because the control process exited with error code. See "systemctl status nginx.service" and "journalctl -xe" for details.</font>
>>
>● nginx.service - nginx - high performance web server
   Loaded: loaded (/usr/lib/systemd/system/nginx.service; disabled; vendor preset: disabled)
   **查看状态**
`[root@izbp1di2a9vvogy93gw0onz local]# systemctl status nginx.service`

 >>  Active: <font color="red">failed </font>(Result: exit-code) since Tue 2017-12-19 10:58:55 CST; 10s ago
     Docs: http://nginx.org/en/docs/
  Process: 26726 ExecStart=/usr/sbin/nginx -c /etc/nginx/nginx.conf (code=exited, status=1/FAILURE)
  Process: 26722 ExecStartPre=/usr/sbin/nginx -t -c /etc/nginx/nginx.conf (code=exited, status=0/SUCCESS)

>>Dec 19 10:58:52 izbp1di2a9vvogy93gw0onz nginx[26726]: nginx: [emerg] **bind() to 0.0.0.0:80 failed** (98: Address already in use)
……
Dec 19 10:58:55 izbp1di2a9vvogy93gw0onz systemd[1]: nginx.service: control process exited, code=exited status=1
Dec 19 10:58:55 izbp1di2a9vvogy93gw0onz systemd[1]: <font color="red">Failed to start nginx - high performance web server.</font>
Dec 19 10:58:55 izbp1di2a9vvogy93gw0onz systemd[1]: Unit nginx.service entered failed state.
Dec 19 10:58:55 izbp1di2a9vvogy93gw0onz systemd[1]: nginx.service failed.

>可以看到是bind to 0.0.0.0:80 fail——绑定80端口失败，
>**查看nginx错误日志（可能要先改log文件权限）**
>`[root@izbp1di2a9vvogy93gw0onz local]# /var/log/nginx/error.log`
>>/var/log/nginx/error.log: line 1: syntax error near unexpected token `('
>>/var/log/nginx/error.log: line 1: `2017/12/19 10:54:17 [emerg] 26705#26705: >><font color="red">bind() to 0.0.0.0:80 failed (98: Address already in use)'</font>
>
>80端口早已被使用，将占用80端口的Tomcat关闭后启动成功
>[root@izbp1di2a9vvogy93gw0onz local]# systemctl start nginx.service

>可用`[root@izbp1di2a9vvogy93gw0onz local]# rpm -ql nginx | grep nginx
`查看安装软件(nginx)的位置信息
![nginx安装位置查看](http://img.blog.csdn.net/20171219114713769?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvY2hlZXRhaGxvdmVy/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)
<table>
<thead>
<tr>
<th>路径</th>
<th>说明</th>
</tr>
</thead>
<tbody>
<tr>
<td>/etc</td>
<td>一些设置文件放置的目录如/etc/crontab</td>
</tr>
<tr>
<td>/usr/bin</td>
<td>一些可执行文件</td>
</tr>
<tr>
<td>/usr/lib</td>
<td>一些程序使用的动态函数库</td>
</tr>
<tr>
<td>/usr/share/doc</td>
<td>一些基本的软件使用手册与帮助文档</td>
</tr>
<tr>
<td>/usr/share/man</td>
<td>一些man page文件</td>
</tr>
</tbody>
</table>

1. /etc/nginx: 默认安装位置；
2. /etc/nginx/conf.d/default.conf： 默认配置信息
3. /etc/nginx/nginx.conf：Nginx 主配置文件
4. /usr/share/nginx/html/：Nginx 的默认站点根目录
5. /etc/nginx/conf.d/default.conf：默认站点配置
6. /var/log/nginx/error.log：错误日志

相关补充：
>启动 Nginx
`systemctl start nginx.service`
 停止 Nginx
`systemctl stop nginx.service`
 重启 Nginx
`systemctl restart nginx.service`
 查看 Nginx 状态
`systemctl status nginx.service`

###*修改Nginx默认配置，转发(映射)请求到不同应用
在Nginx默认配置文件，我的是/etc/nginx/conf.d/default.conf，将不同域名的不同端口配置添加
```
server {

    listen       80;
    server_name  wx.xxx.name;
     location / {
       proxy_pass   http://wx.xxx.name:8080;#对应下面的tomcat中server.xml的端口
    }
}

server {

    listen       80;
    server_name  byxt.xxx.name;
     location / {
       proxy_set_header Host  $host; #设置请求头，并将请求头信息传到服务器，访问服务器文件时不用带上端口号了
       proxy_set_header X-Forwarded-For  $remote_addr;
       proxy_pass   http://byxt.xxx.name:8081;
    }
}
```
如果是统一域名下请求路径不同，则只需要配置一个server，修改location和proxy_pass的配置
```
server{
	…
	…
    #加下面的配置  
    location /路径名1/{  
        proxy_pass http://xxx.xxx.xxx:8080/路径名1;  #主要是这里，这是tomcat1的端口和项目 
        
    location /路径名2/{  
        proxy_pass http://xxx.xxx.xxx:8081/路径名2;  #主要是这里，这是tomcat2的端口和项目  
}
```
重启nginx服务。

###*Tomcat相关配置
先准备两个tomcat，不赘述，之后称tomcat1,tomcat2。
1. 先修改`vi /etc/profile` (*末尾or wherever*)添加
```
#第一个Tomcat
CATALINA_BASE=/usr/local/wechat/apache-tomcat-9.0.0.M18
CATALINA_HOME=/usr/local/wechat/apache-tomcat-9.0.0.M18
export CATALINA_BASE CATALINA_HOME
#第二个Tomcat
CATALINA_2_BASE=/usr/local/tomcat
CATALINA_2_HOME=/usr/local/tomcat
export CATALINA_2_BASE CATALINA_2_HOME
#第一个Tomcat路径
TOMCAT_HOME=/usr/local/wechat/apache-tomcat-9.0.0.M18
export TOMCAT_HOME
#第二个Tomcat路径
TOMCAT_2_HOME=/usr/local/tomcat
export TOMCAT_2_HOME

之后重新执行刚修改的profile使之生效
[root@izbp1di2a]# source /etc/profile
```
这是tomcat启动时catalina.sh中读取实例的路径。
>细分的说CATALINA_HOME 指Tomcat安装路径，CATALINA_BASE 指实例所在位置,CATALINA_HOME 路径下只需要包含 bin 和 lib 目录，而 CATALINA_BASE 只存放 conf、webapps、logs 等这些文件。

随后修改tomcat2的catalina.sh，在
` # OS specific support.  $var _must_ be set to either true or false.`
下面增加如下代码
```
export CATALINA_BASE=$CATALINA_2_BASE
export CATALINA_HOME=$CATALINA_2_HOME
```
不修改的话执行tomcat2的start.sh、shutdown.sh等脚本还是会执行默认的tomcat1的脚本。
2. 修改两个tomcat下的server.xml将http和stop的端口号改为不同，其中的两个http协议端口与nginx默认配置中的xxx.xxx.xxx:8080、xxx.xxx.xxx:8081等对应
3. 如果不成功，检查firewall或者iptables有没有开放对应端口，我就是iptables中没有配置要转发到的8081端口：

```
[root@izbp1di2a9vvogy93gw0onz apache-tomcat-9.0.0.M18]# tail -f /etc/sysconfig/iptables
-A INPUT -p icmp -j ACCEPT
-A INPUT -i lo -j ACCEPT
-A INPUT -p tcp -m state --state NEW -m tcp --dport 22 -j ACCEPT
-A INPUT -m state --state NEW -m tcp -p tcp --dport 80 -j ACCEPT 
-A INPUT -m state --state NEW -m tcp -p tcp --dport 8080 -j ACCEPT
-A INPUT -m state --state NEW -m tcp -p tcp --dport 3306 -j ACCEPT  
-A INPUT -j REJECT --reject-with icmp-host-prohibited
-A FORWARD -j REJECT --reject-with icmp-host-prohibited
COMMIT
# Completed on Sun Nov 26 00:26:06 2017
^[[A^C
[root@izbp1di2a9vvogy93gw0onz apache-tomcat-9.0.0.M18]# vim /etc/sysconfig/iptables
[root@izbp1di2a9vvogy93gw0onz apache-tomcat-9.0.0.M18]# systemctl restart iptables.service
[root@izbp1di2a9vvogy93gw0onz apache-tomcat-9.0.0.M18]# iptables -L -n
Chain INPUT (policy ACCEPT)
target     prot opt source               destination         
ACCEPT     all  --  0.0.0.0/0            0.0.0.0/0            state RELATED,ESTABLISHED
ACCEPT     icmp --  0.0.0.0/0            0.0.0.0/0           
ACCEPT     all  --  0.0.0.0/0            0.0.0.0/0           
ACCEPT     tcp  --  0.0.0.0/0            0.0.0.0/0            state NEW tcp dpt:22
ACCEPT     tcp  --  0.0.0.0/0            0.0.0.0/0            state NEW tcp dpt:80
ACCEPT     tcp  --  0.0.0.0/0            0.0.0.0/0            state NEW tcp dpt:8080
ACCEPT     tcp  --  0.0.0.0/0            0.0.0.0/0            state NEW tcp dpt:8081
ACCEPT     tcp  --  0.0.0.0/0            0.0.0.0/0            state NEW tcp dpt:3306
REJECT     all  --  0.0.0.0/0            0.0.0.0/0            reject-with icmp-host-prohibited

Chain FORWARD (policy ACCEPT)
target     prot opt source               destination         
REJECT     all  --  0.0.0.0/0            0.0.0.0/0            reject-with icmp-host-prohibited

Chain OUTPUT (policy ACCEPT)
target     prot opt source               destination
```