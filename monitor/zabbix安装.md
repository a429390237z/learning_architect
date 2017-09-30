# Zabbix安装 #
## 1.基础环境准备 ##
安装zabbix的yum源。可以通过zabbix的官方源下载，考虑国内速度下载问题，不推荐。可以使用阿里云的镜像，里面有zabbix的Repository。本次安装，系统环境为CentOS 7.1，6.x系列的安装大同小异。

**安装zabbix yum源**
<pre>
//下载yum源rpm包，然后安装
$ cd /usr/local/src
$ wget http://mirrors.aliyun.com/zabbix/zabbix/3.4/rhel/7/x86_64/zabbix-release-3.4-1.el7.centos.noarch.rpm
$ rpm -ivh zabbix-release-3.4-1.el7.centos.noarch.rpm
</pre>

**安装所需要的各种包**

1.server端安装包
<pre>
//注：若是server端不需要监控，则不必要安装zabbix-agent包
$ yum install zabbix-server zabbix-web zabbix-server-mysql zabbix-web-mysql mariadb-server maridb zabbix-agent -y
</pre>

2.agent端安装包</br>
<pre>
$ yum install zabbix-agent
</pre>


**修改PHP时区配置(server端)**

<pre>
$ //修改PHP时区配置
$ sed -i 's@# php_value date.timezone Europe/Riga@php_value date.timezone Asia/Shanghai@g' /etc/httpd/conf.d/zabbix.conf
</pre>

**配置数据库(server端)**

在CentOS7上如果通过yum安装的话，已经找不到mysql，变成了MariaDB。

1.启动数据库
<pre>
$ systemctl start mariadb
</pre>

2.创建zabbix所用的数据库及用户

<pre>
$ mysql
sql> create database zabbix character set utf8 collate utf8_bin;
sql> grant all on zabbix.* to zabbix@'localhost' identified by '123456';
sql> exit
$ cd /usr/share/doc/zabbix-server-mysql-3.4.1
$ zcat create.sql.gz |mysql -uzabbix -p123456 zabbix
</pre>

**修改zabbix配置（server端）**
<pre>
$ vim /etc/zabbix/zabbix_server.conf
DBHost=localhost    #数据库所在主机
DBName=zabbix       #数据库名 
DBUser=zabbix       #数据库用户 
DBPassword=123456   #数据库密码 
</pre>

**启动zabbix和http**
<pre>
systemctl start zabbix-server
systemctl start httpd
</pre>

注意；如果有报错，启动不了，问题可能在内核。yum update后，不用重启，再次启动。

**界面设置**

访问http://192.168.56.11/zabbix/setup.php





