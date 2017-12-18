## 环境部署 ##

###准备安装环境###
Elasticsearch首先需要java环境，所以需要提前安装好JDK，可以直接使用yum安装，也可以从oracle官网下载JDK进行安装，开始之前要确保JDK正常安装并且环境变量也配置正确。<br/>

**安装JDK**
<pre>
$ yum isntall -y java
$ java -version
</pre>

**YUM安装ElasticSearch**
<pre>
// 下载并安装GPG key
$ rpm --import https://packages.elastic.co/GPG-KEY-elasticsearch
// 添加yum仓库
$ vim /etc/yum.repos.d/elasticsearch.repo
[elasticsearch-2.x]
name=Elasticsearch repository for 2.x package
baseurl=http://packages.elastic.co/elasticsearch/2.3/centos
gpgcheck=1
gpgkey=http://packages.elastic.co/GPG-KEY-elasticsearch
enabled=1
// 安装elasticsearch
$ yum install -y elasticsearch
</pre>

**LogStash部署与配置**
和Elasticsearch一样，在开始部署LogStash之前也需要你的环境中正确的安装的JDK。可以下载安装oracle的JDK或者使用yum安装openjdk.（java安装同elasticsearch,略）

**YUM部署LogStash**
<pre>
// 下载并安装GPG key
$ rpm --import https://packages.elastic.co/GPG-KEY-elasticsearch
// 添加yum仓库
$ vim /etc/yum.repos.d/logstash.repo
[logstash-2.3]
name=Logstash repository for 2.3.x packages
baseurl=https://packages.elastic.co/logstash/2.3/centos
gpgcheck=1
gpgkey=https://packages.elastic.co/GPG-KEY=elasticsearch
enabled=1
// 安装logstash
$ yum install -y logstash
</pre>

**kinaba简介**
kibana是为Elasticsearch设计的开源分析和可视化平台，你可以使用kibana来搜索，查看存储在Elasticsearch索引中的数据并与之交互，你可以很容易实现高级的数据分析和可视化，以图表的形式展现出来。


**YUM部署kibana**
<pre>
// 下载并安装GPG key
$ rpm --import https://packages.elastic.co/GPG-KEY-elasticsearch
// 添加yum仓库
$ vim /etc/yum.repos.d/kibana.repo
[kibana-4.5]
name=Kibana repository for 4.5.x packages
baseurl=http://packages.elastic.co/kibana/4.5/centos
gpgcheck=1
gpgkey=http://packages.elastic.co/GPG-KEY-elasticsearch
enabled=1
// 安装kibana
$ yum install -y kibana
</pre>

生产环境可以使用cobbler创建ELKStack仓库
<pre>
$ cobbler repo add --name=logstash-2.3 --mirror=http://packages.elastic.co/logstash/2.3/centos --arch=x86_64 --breed=yum
</pre>


## 使用 ##

查看安装包所包含的文件：</br>
<pre>
$ rpm -ql elasticsearch
$ rpm -ql logstash
$ rpm -ql kibana
</pre>

###1.启用Elasticsearch###
Elasticsearchyum安装的配置文件位于：/etc/elasticsearch,修改elasticsearch.yml,内容如下:
<pre>
// 修改Elasticsearch配置文件
$ vim /etc/elasticsearch/elasticsearch.yml
cluster.name: myes
node.name: myes_node1
path.data: /data/elasticsearch
path.logs: /data/logs/elasticsearch
bootstrap.mlockall: true
network.host: 10.0.0.199
http.port: 9200
// 对目录赋予权限，安装时会创建elasticsearch用户,需要将数据文件和日志文件目录权限赋权限给elasticsearch，否则会报错。
$ chown -R elasticsearch:elasticsearch /data/elasticsearch
$ chown -R elasticsearch:elasticsearch /data/logs/elasticsearch
// 启动elasticsearch
$ /etc/init.d/elasticsearch start
// 可以通过浏览器或者curl访问，-i表示包含请求头信息
$ curl -i http://10.0.0.199:9200
</pre>

**安装插件查看elasticsearch信息，(marvel或head、bigdesk、kopf)**
1.marvel安装
<pre>
// 通过插件方式安装
$ /usr/share/elasticsearch/bin/plugin install license
$ /usr/share/elasticsearch/bin/plugin install marvel-agent
$ /opt/kibana/bin/kibanna plugin --install ../elasticsearch/marvel/2.3.5
// 运行elasticsearch和kibana
$ bin/elasticsearch
$ bin/kibana
// 浏览器访问http://10.0.0.199:5601/app/marvel
</pre>

2.head安装
<pre>
// 通过插件方式安装
$ /usr/share/elasticsearch/bin/plugin install head
// 浏览器访问 http://10.0.0.199:9200/_plugin/head
</pre>

3.kopf安装
<pre>
// 通过插件方式安装
$ /usr/share/elasticsearch/bin/plugin install lmenezes/elasticsearch-kopf
// 浏览器访问：http://10.0.0.199:9200/_plugin/kopf
</pre>



##LogStash的使用##

**默认配置文件存放地点在/etc/logstash/conf.d/**</br>
logstash的启动方式有两种：</br>
1. 命令行启动</br>
<pre>
$ /opt/logstash/bin/logstash -e 'input { stdin{} } output { stdout { codec => rubydebug }elasticsearch { hosts => ["10.0.0.199:9200","10.0.0.4:9200"] index => "logstash-{%YYYY.MM.dd}"}}'
</pre>
2. 配置文件启动</br>
<pre>
$ /opt/logstash/bin/logstash -f /etc/logstash/conf.d/demo.conf
</pre>

logstash执行顺序为：</br>

input --> codec --> filter --> codec --> output</br>
其中codec为再编码阶段，可以理解为转换字符集。

测试logstash配置文件是否正确：
<pre>
$ /opt/logstash/bin/logstash -t -f /etc/logstash/conf.d/nginx.conf
</pre>
logstash启动时较慢，需要查看日志文件看看是否启动成功，日志文件路径为：</br>
/var/log/elasticsearch/elasticsearch.log</br>

**最好修改yum安装logstash的启动脚本，用户改为root，方便对文件进行读取**

处理日志的方式有：</br>
- nginx日志改成json输出
- 文件直接收取
  python脚本读取redis，写成json，写入ES 

实际生产环节中：
1. 每个ES上面启动一个kibana(kibana性能有限)
2. kibana都连自己的ES
3. 前端Nginx'负载均衡、ip_hash + 验证 + ACL

**Rsyslog**<br/>
可以将系统日志收集到ELK中，需要在logstash中的syslog插件中进行收集。配置如下：<br/>
<pre>
input{
    syslog{
        type => "system-log"
        port => 514
    }
}
output{
    elasticsearch{
        hosts => ["10.0.0.199:9200","10.0.0.4:9200"]
        index => "system-rsyslog-%{+YYYY.MM}"
    }
}
</pre>
在需要收集系统日志的机器上修改，/etc/rsyslog.conf,在最后一行添加如下：</br>
<pre>
// 发往的logstash的IP与端口
*.* @@10.0.0.4:514
// 重启rsyslog
systemctl restart rsyslog
</pre>

**通过TCP方式接收日志**<br/>
有时候我们需要补齐漏掉的运行日志，可以在需要补齐的系统上安装logstash，然后配置收集补齐的日志文件路径，但是这样比较麻烦，实际上我们可以在已有的logstash服务器上开启接收指定tcp端口的日志即可。具体配置如下：
<pre>
input{
    tcp{
        type => "tcp"
        port => 6666
        mode => "server"
    }
}
output{
    elasticearch{
        stdout{
            codec => rubydebug
        }
    }
}
// 执行方式有三种：
1.nc管道输出
$ echo "hello" | nc 10.0.0.4 6666
2.读取文件
$ nc 10.0.0.4 6666 < /etc/resolv.conf
3.通过tcp设备文件输入
$ echo "hello" > /dev/tcp/10.0.0.4/6666
</pre>

**filter组件grok**
通过filter组件grok可以对输入进行基于正则表达式的匹配，转换成合适的格式。例子如下:<br/>
默认情况下，apache的日志格式为：</br>
<pre>
10.0.0.1 - - [13/Dec/2017:00:59:30 +0800] "GET / HTTP/1.1" 304 - "-" "Mozilla/5.0 (Windows NT 6.1; Win64; x64; rv:57.0) Gecko/20100101 Firefox/57.0"
</pre>
我们可以在filter中进行过滤：<br/>
<pre>
input{
    stdin{}
}

filter{
    grok{
        match => { "message" => "%{COMBINEDAPACHELOG}" }
    }
}

output{
    stdout{
        codec => rubydebug
    }
}
</pre>
更改后，输出如下：
<pre>
{
        "message" => "10.0.0.1 - - [13/Dec/2017:00:59:30 +0800] \"GET / HTTP/1.1\" 304 - \"-\" \"Mozilla/5.0 (Windows NT 6.1; Win64; x64; rv:57.0) Gecko/20100101 Firefox/57.0\"",
       "@version" => "1",
     "@timestamp" => "2017-12-12T17:21:27.670Z",
           "host" => "cobbler-api-test",
       "clientip" => "10.0.0.1",
          "ident" => "-",
           "auth" => "-",
      "timestamp" => "13/Dec/2017:00:59:30 +0800",
           "verb" => "GET",
        "request" => "/",
    "httpversion" => "1.1",
       "response" => "304",
       "referrer" => "\"-\"",
          "agent" => "\"Mozilla/5.0 (Windows NT 6.1; Win64; x64; rv:57.0) Gecko/20100101 Firefox/57.0\""
}
</pre>


不解耦：<br/>
数据  ==》 logstash  ==》ES<br/>

解耦：<br/>
数据  ==》logstash(可以不是logstash,如flume) ==》MQ(redis)  ==》 logstash(可以不是) ==》ES<br/>

使用redis做MQ：<br/>
前端收集配置：
<pre>
input {
    file {
         path => "/var/log/httpd/access_log"
         start_position => "beginning" 
    }
}
output{
    redis {
        host => "10.0.0.4"
        port => "6379"
        db  => "6"
        data_type => "list"
        key => "demo"
    }
}
</pre>
后端收集配置：
<pre>
input {
    redis {
        host => "10.0.0.4"
        port => "6379"
        db => "6"
        data_type => "list"
        key => "demo"
    }
}

output{
    elasticsearch{
        hosts => ["10.0.0.4:9200"]
        index => "access-log-%{+YYYY.MM.dd}"
    }
}
</pre>


##项目案例规划##

需求分析：<br/>

访问日志：apache访问日志、nginx访问日志、tomcat file - filter
错误日志：error log、java日志、直接收，多行插件处理java日志
系统日志：/var/log/*, syslog  rsylog
运行日志：程序写的  file -> json
网络日志：防火墙、交换机、路由器的日志 syslog

1.标准化：日志放在哪儿(/data/logs/),格式是什么(要求JSON),命名规则，access_log,error_log,runtime_log,日志怎么切割(按天、按小时),access、error crontab进行切分，runtime_log是程序中写的。</br>
所有的原始文本 ---rsync到NAS后，删除最近三天的。</br>
如果使用redis list作为ELKstack的消息队列，需要对list 的key进行监控:llen key。<br/>
深入学习：上INFOQ<br/>
深度实践：数据写入hadoop, webhdfs

2.工具化：如何使用logstash进行收集方案</br>
logstash配置参考：
采集节点shipper.conf：
<pre>
input {
    file{
        path => "/var/log/httpd/access_log"
        start_position => "beginning"
        type => "apache-accesslog"
    }
    file{
        path => "/var/log/elasticsearch/myes.log"
        type => "es-log"
        start_position => "beginning"
        codec => multiline{
            pattern => "^\["
            negate => true
            what => "previous"
        }
    }
    syslog{
        type => "system-syslog"
        port => 514
    }
    
}
output {
    if [type] == "apache-accesslog" {
        redis {
            host => "10.0.0.4"
            port => "6379"
            db => "6"
            data_type => "list"
            key => "apache-accesslog"
        }
    }
    if [type] == "es-log" {
        redis {
            host => "10.0.0.4"
            port => "6379"
            db => "6"
            data_type => "list"
            key => "es-log"
        }
    }
    if [type] == "system-syslog" {
        redis {
            host => "10.0.0.4"
            port => "6379"
            db => "6"
            data_type => "list"
            key => "system-syslog"
        }
    }
 }
</pre>

处理节点
<pre>
input{
    redis {
        host => "10.0.0.4"
        port => "6379"
        db => "6"
        data_type => "list"
        key => "apache-accesslog"
    }
    redis {
        host => "10.0.0.4"
        port => "6379"
        db => "6"
        data_type => "list"
        key => "es-log"
    }
    redis {
        host => "10.0.0.4"
        port => "6379"
        db => "6"
        data_type => "list"
        key => "system-syslog"
    }
}
filter{
    grok {
        if [type] == "apache-accesslog" {
            match = > {"message" => "%{COMBINEDAPACHELOG}"}
        }
    }
}

output{
    if [type] == "apache-accesslog" {
        elasticsearch {
            hosts => ["10.0.0.4:9200","10.0.0.199:9200"]
            index => "apache-accesslog-%{+YYYY.MM.dd}"
        }
    }
    if [type] == "es-log" {
        elasticsearch {
            host => ["10.0.0.4:9200","10.0.0.199:9200"]
            index => "es-log-%{+YYYY.MM}"
        }
    }
    if [type] == "system-syslog" {
        elasticsearch {
            hosts => ["10.0.0.4:9200","10.0.0.199:9200"]
            index => "system-syslog-%{+YYYY.MM}"
        }
    }
}

</pre>
