Saltstack 是用Python写的，提供API接入。<br/>
salt参考：[https://www.unixhot.com/docs/saltstack](https://www.unixhot.com/docs/saltstack)

三大功能：远程执行</br>
          配置管理(状态)</br>
          云管理

Puppet: ruby写的。</br>
ansible：使用Python编写。

四种运行方式：

- Local
- Minion/Master(C/S)
- Syndic(zabbix proxy)
- Salt SSH

远程仓库：[http://repo.saltstack.com/](http://repo.saltstack.com/)


salt防火墙配置：<br/>
<pre>
# Allow Minions from these networks
-I INPUT -s 10.1.2.0/24 -p tcp -m multiport --dports 4505,4506 -j ACCEPT
-I INPUT -s 10.1.3.0/24 -p tcp -m multiport --dports 4505,4506 -j ACCEPT
# Allow Salt to communicate with Master on the loopback interface
-A INPUT -i lo -p tcp -m multiport --dports 4505,4506 -j ACCEPT
# Reject everything else
-A INPUT -p tcp -m multiport --dports 4505,4506 -j REJECT
</pre>

作用：

1. 远程执行：salt '*' cmd.run 'uptime'</br>
             salt '*' test.ping</br>
2. State，需要写一个文件。格式：YAML，后缀：.sls

   YAML:三板斧</br>
   1. 缩进(2个空格，不能使用Tab键)
   2. 冒号(冒号处理键值对的时候，需要有一个空格)
   3. 短横线(后面有一个空格)

安装：要是重装salt时，一定要记得清除缓存，master key缓存：/etc/salt/pki/master/，minion key缓存：/etc/salt/pki/minion/

<pre>
//master端：
$ yum install -y salt-master
//minion端：
$ yum install -y salt-minion
//minion端修改:
  ====> master: 10.0.0.199
//master端修改：
  ====> 
file_roots:
  base:
    - /srv/salt
//master端收编minion
$ salt-key -A
</pre>


state状态文件示例：

![](https://i.imgur.com/5jlRCV9.png)

apache.sls文件如下：
<pre>
apache-install:
  pkg.installed:
    - names:
      - httpd
      - httpd-devel
   
apache-service:
  service.running:
    - name: httpd
    - enable: True
</pre>

执行状态模块：salt '*' state.sls web.apache

入口文件默认为top.sls，路径在base目录下，即/srv/salt目录下。</br>
top.sls示例如下：
<pre>
base:
  'cobbler-api-test':
    - web.apache
  'zabbix-server':
    - web.apache
</pre>

执行top文件：salt '*' state.highstate</br>
执行前需先测试：salt '*' state.highstate test='True'


saltstack为什么快：</br>
使用zeroMQ，使用publish/subscript系统，创建监听4505的端口(全部都是长连接)，使用4506端口进行数据传输。

## Saltstack数据系统 ##
- Grains(谷粒)
- Pillar(柱子)

### Grains ###
Grains: 静态数据，当minion启动的时候收集的minion本地的相关信息。(操作系统版本，内核版本，CPU，内存，硬盘，设备型号，序列号)

1. 资产管理，信息查询。
2. 用于目标选择。
3. 配置管理中使用。

使用Grains获取服务器信息：
<pre>
//列出所有的key
$ salt '*' grains.ls
//获取所有的items
$ salt '*' grains.items
//获取单个item
$ salt '*' grains.item os
//通过item执行命令
$ salt -G 'os:CentOS' cmd.run 'uptime'
 </pre>

自定义item:</br>
- 在/etc/salt/minion中,搜索grain，定义Grains.
- 在文件/etc/salt/grains中定义。

修改完后，需要重启minion端生效：systemctl restart salt-minion</br>
不重启生效的方法，刷新：salt '*' saltutil.sync_grains

TOP File使用案例：

base:
  'zabbix-server':
    - web.apache
   'roles:apache':
    - match: grain
    - web.apache


配置管理案例：

开发一个grains,需要在master端，/srv/salt目录下创建_grains目录。</br>
<pre>
//在_grains目录下，创建文件my_grains.py:
#!/usr/bin/env python
#-*- coding: utf-8 -*-

def my_grains():
    #初始化一个grains字典
    grains = {}
    #设置字典中的key-value
    grains['laas'] = 'openstack'
    grains['edu'] = 'oldboyedu'
    #返回这个字典
    return grains
</pre>

刷新Grains：salt '*' saltutil.sync_grains</br>
Grains优先级：

1. 系统自带
2. Grains文件写的
3. minion配置文件写的
4. 自己写的

### Pillar ###

pillar是动态的，给特定的minion指定特定的数据，只有指定的minion自己能看到自己的数据。类似于top file。<br/>

1 . 写pillar sls文件(jinjia2),/srv/pillar/web/apache.sls
<pre>
hehe:
  {% if grains['os'] == 'CentOS' %}
  apache: httpd
  {% elif grains['os'] == 'Debian' %}
  apache: apache2
  {% endif %}
</pre>
2 . 写top file。(top.sls)


pillar使用场景：<br/>

1.目标选择<br/>
Grains VS Pillar<br/>
<pre>
           类型        数据采集方式        应用场景
Grains     静态        minion启动时收集    数据查询，目标选择，配置管理
Pillar     动态        master自定义        目标选择，配置管理，敏感数据
</pre>


##深入学习saltstack远程执行##
<pre>
salt '*' cmd.run 'w'
</pre>

命令：salt<br/>
目标：'*'<br/>
模块：cmd.run 自带150+模块，可自定义模块<br/>
返回：执行后结果返回。Returnners<br/>

*目标：Targeting*

两种：一种和minion ID有关,一种和minion ID无关。<br/>
所有匹配目标的方式，都可以用到top file里面来指定目标<br/>

1.minion ID有关的方法：<br/>
<pre>
    1.minion ID
    2.通配符
    3.列表方式，指定 -L参数，逗号分隔
    4.正则表达式。指定 -E参数
</pre>

2.minion ID无关的方法：<br/>
<pre>
    1.子网、IP地址
</pre>

**匹配方式总结**
<pre>
Letter       Type              Example
G            Grains glob       G@os:Ubuntu
E            PCRE Minion ID    E@web\d+\.(dev|qa|prod)\.Ioc
P            Grains PCRE       P@os:(RedHat|Fedora|CentOS)
L            List of minion    L@node1,node2,node3
I            Pillar glob       I@pdata:foobar
J            Pillar PCRE       J@pdata:(foo|bar)$
S            Subnet/IP address S@192.168.1.0/24 or S@192.100.1.12
R            Range cluster     R@foo,bar
C            Mix
N            group               
</pre>

主机名设置方案:<br/>
<pre>
1. IP地址
2. 根据业务来进行设置，如redis-node1-redis04-idc04.soa.example.com
   redis-node1：redis第一个节点
   redis04：集群
   idc04：机房
   soa：业务线
</pre>
      

**配置RETURNER**
1.执行salt后的命令不仅可以在master端显示，还可以通过 --return后面指定的参数，返回给对应的接收端。如：<br/>
<pre>
//不仅将salt执行命令后的结果显示在终端上，还将结果保存至mysql数据库
salt '*' test.ping --return mysql
</pre>
mysql数据库的配置：

1. 安装mysql数据库，minion端需要安装MySQL-python.
2. 建库.
<pre>
CREATE DATABASE  `salt`
  DEFAULT CHARACTER SET utf8
  DEFAULT COLLATE utf8_general_ci;

USE `salt`;

--
-- Table structure for table `jids`
--

DROP TABLE IF EXISTS `jids`;
CREATE TABLE `jids` (
  `jid` varchar(255) NOT NULL,
  `load` mediumtext NOT NULL,
  UNIQUE KEY `jid` (`jid`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8;
CREATE INDEX jid ON jids(jid) USING BTREE;

--
-- Table structure for table `salt_returns`
--

DROP TABLE IF EXISTS `salt_returns`;
CREATE TABLE `salt_returns` (
  `fun` varchar(50) NOT NULL,
  `jid` varchar(255) NOT NULL,
  `return` mediumtext NOT NULL,
  `id` varchar(255) NOT NULL,
  `success` varchar(10) NOT NULL,
  `full_ret` mediumtext NOT NULL,
  `alter_time` TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  KEY `id` (`id`),
  KEY `jid` (`jid`),
  KEY `fun` (`fun`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8;

--
-- Table structure for table `salt_events`
--

DROP TABLE IF EXISTS `salt_events`;
CREATE TABLE `salt_events` (
`id` BIGINT NOT NULL AUTO_INCREMENT,
`tag` varchar(255) NOT NULL,
`data` mediumtext NOT NULL,
`alter_time` TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
`master_id` varchar(255) NOT NULL,
PRIMARY KEY (`id`),
KEY `tag` (`tag`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8;
</pre>
3 . 创建用户，并授权
<pre>
grant ALL on salt.* to 'salt'@'%' identified by '@Wang1234';
flush privileges;
</pre>
4 .分别在minion的配置文件中添加如下配置：<br/>
<pre>
mysql.host: '10.0.0.4'
mysql.user: 'salt'
mysql.pass: '@Wang1234'
mysql.db: 'salt'
mysql.port: 3306
</pre>
5 .重启minion，master端执行salt命令<br/>
<pre>
systemctl restart salt-minion
salt '*' test.ping --return mysql
</pre>
6 .数据库中出现数据
![](https://i.imgur.com/pvZG9AO.png) 

**自定义模块**

1. 在master配置文件中开启 /srv/salt.
2. 创建/srv/salt/_modules目录，在目录中创建my_disk.py
<pre>
#!/usr/bin/env python
# -*- coding: utf-8 -*-

def list():
   cmd = 'df -h'
   ret = __salt__['cmd.run'](cmd)
   return ret
</pre>
3 .刷新，salt '*' saltutil.sync_modules,实际上是将py文件同步到minion端的/var/cache/salt/minion/extmods/modules下。<br/>
4 .执行salt '*' my_disk.list