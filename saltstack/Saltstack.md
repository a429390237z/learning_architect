Saltstack 是用Python写的，提供API接入。

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


作用：

1. 远程执行：salt '*' cmd.run 'uptime'</br>
             salt '*' test.ping</br>
2. State，需要写一个文件。格式：YAML，后缀：.sls

   YAML:三板斧</br>
   1. 缩进(2个空格，不能使用Tab键)
   2. 冒号(冒号处理键值对的时候，需要有一个空格)
   3. 短横线(后面有一个空格)

安装：

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

pillar是动态的，给特定的minion指定特定的数据，只有指定的minion自己能看到自己的数据。