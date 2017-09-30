# zabbix使用完整流程 & zabbix生产案例实战 #

- 添加用户组时需要添加权限，权限只能按用户组分配。
- 创建用户，选择用户角色。
- 设置好报警媒介。
- 配置Action。添加新主机后，要确认权限分配。

**两台主机配置连线实时流量情况**
<pre>
{hostname:net.if.in[eth0].last(0)}
</pre>

## zabbix生产案例实战 ##

1. 项目规划

 - 主机分组：
   - 交换机
   - Nginx
   - Tomat
   - mysql
 - 监控对象识别：
   1. 使用SNMP监控交换机
   2. 使用IPMI监控服务器硬件
   3. 使用Agent监控服务器
   4. 使用JMX监控Java
   5. 监控Mysql
   6. 监控Web状态
   7. 监控Nginx状态


**1.1 交换机上开启snmp**
<pre>
config t
snmp-server community public ro
end
</pre>

**使用GNS3模拟交换机**

**1.2 在zabbix上添加监控**

设置SNMP interfaces，关联监控模版

IPMI：</br>
  zabbix监控IPMI经常获取不到或者超时。</br>
  建议。使用自定义item，本地执行ipmitool命令获取数据。

**1.3 zabbix JMX配置**

  使用Zabbix Java Gateway代理。</br>
  关系：Zabbix-Server ==> Zabbix-Java-Gateway ==> java application

<pre>
//安装zabbix-java-gateway时默认安装java-1.8.0
$ yum install -y zabbix-java-gateway java-1.8.0
//配置
$ vim /etc/zabbix/zabbix_java_gateway.conf
  # LISTEN_IP="0.0.0.0"
  # LISTEN_PORT=10052
  # START_POLLERS=5
  # TIMEOUT=3
//启动
$ systemctl start zabbix-java-gateway.service
//给zabbix-server指定java-gate-way位置
$ vim /etc/zabbix/zabbix_server.conf
  JavaGateway= 10.0.0.34
  #JavaGatewayPort=10052
  StartJavaPollers=5
//重启zabbix-server
$ systemctl restart zabbix-server
</pre>

**安装以及配置tomcat**
<pre>
//下载tomcat
$ cd /usr/local/src
$ wget http://mirrors.shuosc.org/apache/tomcat/tomcat-8/v8.0.46/bin/apache-tomcat-8.0.46.tar.gz
$ tar xf apache-tomcat-8.0.46.tar.gz
$ mv apache-tomcat-8.0.46 ../
$ ln -s apache-tomcat-8.0.46 tomcat
//开启JMX远程监控
$ vim /usr/local/tomcat/bin/catalina.sh
CATALINA_OPTS="$CATALINA_OPTS -Dcom.sun.management.jmxremote
-Dcom.sun.management.jmxremote.port=12345
-Dcom.sun.management.jmxremote.ssl=false
-Dcom.sun.management.jmxremote.authenticate=false
-D java.rmi.server.hostname=10.0.0.137"
//启动tomcat
/usr/local/tomcat/bin/startup.sh
</pre>

- 手动检测监控状态</br>
<pre>
$ yum install -y zabbix-get
</pre>

**命令行自动补全**

<pre>
$ yum install -y bash-completion
</pre>

### nginx 监控 ###

1. 开启Nginx监控
2. 编写脚本进行数据采集
3. 设置用户自定义参数
4. 重启zabbix-agent
5. 添加item
6. 创建图形
7. 创建触发器
8. 创建模版

*注意：需要配置nginx_status,nginx.conf下设置*

<pre>
 location /nginx_status {
            stub_status on;
            access_log off;
            allow 127.0.0.1;
            deny all;
}
</pre>

- 编写脚本。
- 上传到/etc/zabbix/zabbix_agentd.d
- 修改agent配置Include=/etc/zabbix/zabbix_agentd.d/*.conf
- 添加执行权限: chmod +x linux_server_status.sh
- 测试脚本：./linux_server_status.sh nginx_status 80 Active
- 配置agent端，在/etc/zabbix/zabbix_agentd.d下创建文件，nginx_status.conf，内容如下:
<pre>
UserParameter=Nginx_status[*],/etc/zabbix/zabbix_agentd.d/linux_server_status.sh $1 $2 $3
</pre>
- 重启agent：systemctl restart zabbix-agent
- zabbix-server端测试：zabbix-get -s 10.0.0.137 -k Nginx.Status
- 创建模版，添加items，创建图形
- 主机选择模版


###  自定义发送邮件脚本 ###

**自定义告警脚本**

1. 放在/usr/lib/zabbix/alertscripts
2. 要支持三个参数：1.收件人；2.主题；3.内容
3. 执行权限
4. web界面添加
5. 修改Actions


### 监控瓶颈 ###

1. 监控主机多，性能跟不上，延迟大
2. 多机房，防火墙

zabbix轻松解决，Nagios不太好解决。

zabbix有两种模式：

- 被动模式
- 主动模式

当监控主机超过300+，建议使用主动模式

主动模式配置：</br>
修改/etc/zabbix/zabbix_agentd.conf:</br>
==> StartAgents=0
==> ServerActive=10.0.0.34
==> Hostname=zabbix-server

system restart zabbix-agent

### Zabbix Proxy ###

zabbix-server ==> zabbix proxy ==> zabbix-agent

使用场景：</br>
1. 多机房监控
2. 机器数量太多

配置：</br>
<pre>
//安装yum包
$ yum install -y zabbix-proxy zabbix-proxy-mysql mariadb-server
//启动mysql数据库
$ systemctl start mariadb
//创建库，用户
sql> create database zabbix_proxy character set utf8;
sql> grant all on zabbix_proxy.* to zabbix_proxy@localhost identified by 'zabbix_proxy';
//生成建表语句
$ cd /usr/share/doc/zabbix-proxy-mysql-3.0.3/
$ zcat shema.sql.gz | mysql -uzabbix_proxy -pzabbix_proxy zabbix_proxy
//修改zabbix_proxy配置
$ vim /etc/zabbix/zabbix_proxy.conf
  ====> Server=10.0.0.34
  ====> ServerActive=10.0.0.34
  ====> Hostname=zabbix-proxy
  ====> DBHost=localhost
  ====> DBUser=zabbix_proxy
  ====> DBPassword=zabbix_proxy
//Web界面添加
Administration ==》 Proxies ==》 Create proxy
//添加需要被管理的agent，修改agent配置/etc/zabbix/zabbix_agentd.conf:
  ====> Server=10.0.0.199
  ====> ServerActive=10.0.0.199
</pre>


### zabbix 自动化监控 ###

1. 自动注册
    1.1 zabbix agent自动添加

2. 主动发现
    2.1 自动发现Discover
    2.2 zabbix api

**zabbix api**

通过zabbix api实现自动化查询，自动化管理

参考文档：[https://www.zabbix.com/documentation/3.4/zh/manual/api](https://www.zabbix.com/documentation/3.4/zh/manual/api)
