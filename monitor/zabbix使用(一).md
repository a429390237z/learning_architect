# zabbix使用 #

**自定义监控项**

1. 添加用户自定义参数

<pre>
$ cat /etc/zabbix/zabbix_agentd.d/nginx.conf
UserParameter=nginx.active,/usr/bin/curl -s "http://10.0.0.137:8080/nginx_status"|awk '/Active/{print $NF}'
</pre>

2. 重启zabbix-agent
3. 在server端使用zabbix-get测试获取。</br>
zabbix-get命令安装:

<pre>
$ yum install -y zabbix-get
</pre>

测试获取情况：

<pre>
$ zabbix-get -s 10.0.0.137 -p 10050 -k "nginx.active"
</pre>

4. 在web界面创建item

5. 自定义图形

6. 自定义map

7. 自定义screen

作业：

1. 网络监控：Smokeping部署
2. zabbix乱点
3. 下次分享：Piwik 流量分析系统
