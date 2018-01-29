#salt配置管理实战#

##基于haproxy、nginx+php、mysql、memecached的架构##

![](https://i.imgur.com/aXFfXiJ.png)
头脑风暴：<br/>
1. 系统初始化
2. 功能模块： 设置单独的目录 haproxy、nginx、php、mysql、memecached<br/>
   尽可能的全、独立<br/>
3. 业务模块：根据业务员类型划分，例如web服务，论坛bbs<br/>
   include<br/>

架构图如下：<br/>
 

干活:<br/>
  1.salt环境配置<br/>
  开发、测试(功能测试环境、性能测试环境)、预生产、生产<br/>

1.base 基础环境<br/>
2.prod 生产环境<br/>

继续学习状态间关系：<br/>

1. unless: 如果unless后面的命令返回为True，那么就不执行。
2. onlyif

haproxy下载地址：<br/>
[http://download.chinaunix.net/download/0013000/12508.shtml](http://download.chinaunix.net/download/0013000/12508.shtml)

默认salt状态执行时，只执行base环境，要执行其它环境需要制定saltenv='prod'<br/>