#配置管理#

Salt State，SLS描述文件，YAML格式。
名称ID，默认是内部申明。
<pre>
apache-install:
  pkg.installed:
    - names:
      - httpd
      - httpd-devel

apache-service: # ID声明，高级状态，ID必须唯一
  service.running: # 状态声明
    - name: httpd # 选项声明
    - enable: True

php:
  pkg.installed
</pre>

LAMP架构

1.安装软件包（pkg）<br/>
2.修改配置文件(file)<br/>
3.启动服务（service）<br/>

<pre>
pkg.installed #安装
pkg.latest #确保最新版本
pkg.update #升级
pkg.purge #卸载并删除配置文件
pkg.removed #卸载

一个ID声明下面，状态模块不能重复使用。

#同时安装多个包
common_packages:
  pkg.installed:
    - pkgs:
      - unzip
      - dos2unix
      - salt-minion: 2015.8.5-1.el6

// 安装lamp第一种方式：lamp.sls
// salt://表示当前环境的根目录，例如：
file_roots:
  base:
    - /srv/salt

==========================================
lamp-pkg:
  pkg.installed:
    - pkgs:
      - httpd
      - php
      - mariadb
      - mariadb-server
      - php-mysql
      - php-cli
      - php-mbstring

apache-config:
  file.managed:
    - name: /etc/httpd/conf/httpd.conf
    - source: salt://files/httpd.conf
    - user: root
    - group: root
    - mode: 644

php-config:
  file.managed:
    - name: /etc/php.ini
    - source: salt://files/php.ini
    - user: root
    - group: root
    - mode: 644

mysql-config:
  file.managed:
    - name: /etc/my.cnf
    - source: salt://files/my.cnf
    - user: root
    - group: root
    - mode: 644
    - require_in:
      - service: mysql-service

apache-service:
  service.running:
    - name: httpd
    - enable: True
    - reload: True
    - require:
      - pkg: lamp-pkg
    - watch:
      - file: apache-config
// 如果apache-config这个id的状态发生变化就reload
// 如果不加reload: True，那么就restart

mysql-service:
  service.running:
    - name: mariadb
    - enable: True
    - reload: True
===========================================================

安装lamp第二种方式：lamp.sls

apache-server:
  pkg.installed:
    - pkgs:
      - httpd
      - php
      - php-mysql
      - php-cli
      - php-mbstring
  file.managed:
    - name: /etc/httpd/conf/httpd.conf
    - source: salt://lamp/files/httpd.conf
    - user: root
    - group: root
    - mode: 644
  service.running:
    - name: httpd
    - enable: True
    - reload: True

mysql-server:
  pkg.installed:
    - pkgs:
      - mariadb
      - mariadb-server
  file.managed:
    - name: /etc/my.cnf
    - source: salt://lamp/files/my.cnf
    - user: root
    - group: root
    - mode: 644

php-config:
  file.managed:
    - name: /etc/php.ini
    - source: salt://lamp/files/php.ini
    - user: root
    - group: root
    - mode: 644

=========================================================
</pre>

状态间关系：<br/>

1. 我依赖谁 require <br/>
<pre>
   requre:
   - pkg: lamp-pkg
   - file: apache-config
</pre>
2. 我被谁依赖 require_in <br/>
3. 我监控谁 watch <br/>
4. 我被谁监控 watch_in <br/>
5. 我引用谁,类似于python的模块导入 <br/>
   include:<br/>
     \- lamp.pkg
6. 我扩展谁 <br/>


##如何编写SLS技巧##

1. 按状态分类，如果单独使用，很清晰
2. 按服务分类，可以被其他的sls include。例如LNMP include mysql的服务

### yaml -------jinja ###

两种分隔符: {%...%} 和 {{...}}，其中{%...%}代表表达式，{{...}}代表变量<br/>
3步走：
<pre>
1. 告诉File模块，你要使用jinja
   template: jinja
2. 你要列出参数列表
   - defaults:
     PORT: 88
3. 模板引用
   {{ PORT }}
模板里面支持：salt grains pillar 进行赋值
1. 写在模板文件中：
Grains: Listen {{ grains['fqdn_ip4'][0] }}:{{PORT}}
salt远程执行模块,例如获取eth0的mac地址：{{ salt['network.hw_addr']('eth0') }}
Pillar:  {{ pillar['hehe']['apache'] }}
2. 写在SLS里面的Defaults,变量列表中。
- default:
  IPADDR: {{ grains['fqdn_ip4'][0] }}
  PORT: 88
</pre>

所有的minion出去的pillar中item rsyslog的值是server的minion

参考salt配置：[https://github.com/saltstack-formulas](https://github.com/saltstack-formulas)