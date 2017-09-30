# 使用Percona监控插件监控MySQL #

相关安装文档：[https://www.percona.com/doc/percona-monitoring-plugins/LATEST/zabbix/index.html](https://www.percona.com/doc/percona-monitoring-plugins/LATEST/zabbix/index.html)

- 操作步骤：

1. 安装REPO源
<pre>
$ yum install http://www.percona.com/downloads/percona-release/redhat/0.1-4/percona-release-0.1-4.noarch.rpm 
</pre>

2. 安装所需yum包
<pre>
$ yum install percona-zabbix-templates php php-mysql
</pre>

  1)php脚本用来数据采集
  2)shell,调用php
  3)zabbix配置文件
  4)zabbix模版文件

  创建zabbix监控专用用户。

