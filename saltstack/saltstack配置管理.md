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

#同时安装多个包
common_packages:
  pkg.installed:
    - pkgs:
      - unzip
      - dos2unix
      - salt-minion: 2015.8.5-1.el6

lamp-pkg:
  pkg.installed:
    - pkgs:
      - httpd
      - php
      - mysql
      - mysql-server
      - php-mysql
      - php-cli
      - php-mbstring
</pre>