#DNS域名服务器#
1.DNS命令行查看工具：host，nslookup，dig<br/>
2.如果有防火墙限制，tcp、udp 53端口都要开.<br/>

安装方式：1.yum安装(yum install bind bind-devel bind-utils bind-chroot -y)<br/>
2.源码安装<br/>

配置参考：
<pre>
DNS安装脚本YUM版
#!/bin/bash
####################################################################
# Auto install bind
# Create Date :  2012-11-28
# Written by :shanks
# Organization:  DangDang
####################################################################

IN_Face=`route -n |awk '{if($4~/UG/){print $8}}'|head -n 1`
Local_IP=`/sbin/ifconfig|grep -B1 -C1 -w "${IN_Face}"|grep -w 'inet addr'|awk -F: '{print $2}'|awk '{print $1}'`

cd /usr/local/src/
yum -y install bind-utils bind bind-devel bind-chroot bind-libs >>/tmp/init_sn.log -y && rndc-confgen -r /dev/urandom -a || exit 1
  # ***config /etc/named.conf***
cat << shanks1  > /etc/named.conf
options {
  version "1.1.1";
  listen-on port 53 {any;};
  directory "/var/named/chroot/etc/";
  pid-file "/var/named/chroot/var/run/named/named.pid";
  allow-query { any; };
  Dump-file "/var/named/chroot/var/log/binddump.db";
  Statistics-file "/var/named/chroot/var/log/named_stats";
  zone-statistics yes;
  memstatistics-file "log/mem_stats";
  empty-zones-enable no;
  forwarders {202.106.196.115;8.8.8.8; };
};

key "rndc-key" {
        algorithm hmac-md5;
        secret "Eqw4hClGExUWeDkKBX/pBg==";
};

controls {
       inet 127.0.0.1 port 953
               allow { 127.0.0.1; } keys { "rndc-key"; };
 };

logging {
  channel warning {
    file "/var/named/chroot/var/log/dns_warning" versions 10 size 10m;
    severity warning;
    print-category yes;
    print-severity yes;
    print-time yes;
  };
  channel general_dns {
    file "/var/named/chroot/var/log/dns_log" versions 10 size 100m;
    severity info;
    print-category yes;
    print-severity yes;
    print-time yes;
  };
  category default {
    warning;
  };
  category queries {
    general_dns;
  };
};

include "/var/named/chroot/etc/view.conf";

shanks1
# ***config /etc/rndc.key***
cat << shanks2  > /etc/rndc.key
key "rndc-key" {
        algorithm hmac-md5;
        secret "Eqw4hClGExUWeDkKBX/pBg==";
};
shanks2
# ***config /etc/rndc.conf***
cat << shanks3  > /etc/rndc.conf
# Start of rndc.conf
key "rndc-key" {
        algorithm hmac-md5;
        secret "Eqw4hClGExUWeDkKBX/pBg==";
};

options {
        default-key "rndc-key";
        default-server 127.0.0.1;
        default-port 953;
};
shanks3
# ***config /var/named/chroot/etc/view.conf***
cat << shanks4  > /var/named/chroot/etc/view.conf
view "View" {
             allow-transfer {
                #dns-ip-list; 
        };      
             notify  yes;
             also-notify {
                #dns-ip-list; 
        };
       
#  ixfr-from-differences yes;
zone "com" {
        type    master;
        file    "com.zone";
        allow-transfer {
                10.255.253.211;
        };
        notify  yes;
        also-notify {
                10.255.253.211;
        };
  };
        zone "forward.com" {
             type    forward;
              forwarders { 10.255.253.220; };
        };
};
shanks4
# ***config  /var/named/chroot/etc/com.zone***
cat << shanks5  >  /var/named/chroot/etc/com.zone
\$ORIGIN .
\$TTL 3600       ; 1 hour
com                  IN SOA  op.shanks.com. dns.shanks.com. (
                                2000       ; serial
                                900        ; refresh (15 minutes)
                                600        ; retry (10 minutes)
                                86400      ; expire (1 day)
                                3600       ; minimum (1 hour)
                                )
                        NS      op.shanks.com.
\$ORIGIN com.
shanks              A       1.2.3.4
shanks5
cd /var && chown -R named.named named/
/etc/init.d/named start
chkconfig named on
#check install status.
check_cmd=`host  -s -W 0.5 shanks.com 127.0.0.1|grep "1.2.3.4"`
if [ -z "${check_cmd}" ]
then
  echo "<ERROR!> hey,man.install bind --- ERROR!"
  exit 5
else
  echo "<OK> hey,man.install bind --- ok."
  rndc stats
fi

if [ -f /tmp/Install_bind.sh ]
then
  rm -rf /tmp/Install_bind.sh
fi
DNS安装脚本9.9版（编译版本）
#!/bin/bash
####################################################################
# Auto install bind
# Create Date :  2012-11-28
# Written by :shanks
# Organization:  DangDang
####################################################################

IN_Face=`route -n |awk '{if($4~/UG/){print $8}}'|head -n 1`
Local_IP=`/sbin/ifconfig|grep -B1 -C1 -w "${IN_Face}"|grep -w 'inet addr'|awk -F: '{print $2}'|awk '{print $1}'`

prefix='/usr/local/bind'

cd /usr/local/src/ && wget http://192.168.1.9/soft/dns/9.9/bind-9.9.7-P2.tar.gz && tar zxf bind-9.9.7-P2.tar.gz
if [ -d '/usr/local/src/bind-9.9.7-P2' ]
then
  cd /usr/local/src/bind-9.9.7-P2 && ./configure --prefix=/usr/local/bind --enable-threads --with-libtool && make && make install
  REAV=$?
  if [ ${REAV} != 0 ]
  then
    echo 'bind make faild!!!'
    exit 2
  fi
else
  echo 'bind src get filed!!!'
  exit 1
fi
  # ***config /etc/named.conf***
cat << shanks1  > ${prefix}/etc/named.conf
options {
  version "1.1.1";
  listen-on port 53 {any;};
  directory "${prefix}/etc/";
  pid-file "${prefix}/var/run/named.pid";
  allow-query { any; };
  Dump-file "${prefix}/var/binddump.db";
  Statistics-file "${prefix}/var/named_stats";
  zone-statistics yes;
  memstatistics-file "var/mem_stats";
  empty-zones-enable no;
  masterfile-format text;
#  allow-update {none;}; 
#  allow-recursion {any;}; 
#  serial-query-rate 100;
#  recursion no;
#  dnssec-enable yes;
};

key "rndc-key" {
        algorithm hmac-md5;
        secret "Eqw4hClGExUWeDkKBX/pBg==";
};

controls {
       inet 127.0.0.1 port 953
               allow { 127.0.0.1; } keys { "rndc-key"; };
 };

logging {
  channel warning {
    file "${prefix}/var/dns_warning" versions 10 size 10m;
    #file "${prefix}/var/dns_warning";
    severity warning;
    print-category yes;
    print-severity yes;
    print-time yes;
  };
  channel general_dns {
    file "${prefix}/var/dns_log" versions 10 size 50m;
    #file "${prefix}/var/dns_log";
    severity info;
    print-category yes;
    print-severity yes;
    print-time yes;
  };
  category default {
    warning;
  };
  category queries {
    general_dns;
  };
};

include "${prefix}/etc/view.conf";

shanks1
# ***config /etc/rndc.key***
cat << shanks2  > /etc/rndc.key
key "rndc-key" {
        algorithm hmac-md5;
        secret "Eqw4hClGExUWeDkKBX/pBg==";
};
shanks2
# ***config /etc/rndc.conf***
cat << shanks3  > ${prefix}/etc/rndc.conf
# Start of rndc.conf
key "rndc-key" {
        algorithm hmac-md5;
        secret "Eqw4hClGExUWeDkKBX/pBg==";
};

options {
        default-key "rndc-key";
        default-server 127.0.0.1;
        default-port 953;
};
shanks3
# ***config ${prefix}/etc/view.conf***
cat << shanks4  > ${prefix}/etc/view.conf
view "View" {
             allow-transfer {
                #dns-ip-list; 
        };      
             notify  yes;
             also-notify {
                #dns-ip-list; 
        };
       
#  ixfr-from-differences yes;
zone "com" {
        type    master;
        file    "com.zone";
        allow-transfer {
                10.255.253.211;
        };
        notify  yes;
        also-notify {
                10.255.253.211;
        };
  };
        zone "forward.com" {
             type    forward;
             forwarders { 10.255.253.220; };
        };
};
shanks4
# ***config  ${prefix}/etc/com.zone***
cat << shanks5  >  ${prefix}/etc/com.zone
\$ORIGIN .
\$TTL 3600       ; 1 hour
com                  IN SOA  op.shanks.com. dns.shanks.com. (
                                2000       ; serial
                                900        ; refresh (15 minutes)
                                600        ; retry (10 minutes)
                                86400      ; expire (1 day)
                                3600       ; minimum (1 hour)
                                )
                        NS      dns.shanks.com.
\$ORIGIN com.
shanks              A       1.2.3.4
shanks5
useradd named -s /sbin/nologin
cd /usr/local && chown -R named.named bind/
if [ -f /etc/init.d/named ]
then
  rm -rf /etc/init.d/named
fi
wget -q http://192.168.1.9/soft/dns/9.9/named -O /etc/init.d/named && chmod +x /etc/init.d/named
/etc/init.d/named start
ln -s ${prefix}/sbin/rndc /usr/bin/rndc
ln -s ${prefix}/bin/host /usr/bin/host
ln -s ${prefix}/bin/dig /usr/bin/dig
chkconfig named on
#check install status.
check_cmd=`host  -s -W 0.5 shanks.com 127.0.0.1|grep "1.2.3.4"`
if [ -z "${check_cmd}" ]
then
  echo "<ERROR!> hey,man.install bind --- ERROR!"
else
  echo "<OK> hey,man.install bind --- ok."
fi

if [ -f /tmp/Install_bind.sh ]
then
  rm -rf /tmp/Install_bind.sh
fi
</pre>

**DNS主从同步配置**<br/>
DNS从服务器上其它配置与主服务器基本一致，不一致的地方如下：<br/>
<pre>
// /var/named/chroot/etc/view.conf
view "SlaveView" {
  zone "shanks.com" {
        type    slave;
        #',' as deliter
        masters {10.0.0.4;};
        file    "slave.shanks.com.zone";
  };
};
// 不用编辑 slave.shanks.com.zone，通过rndc reload，此配置文件会被加载到slave下。
// 主服务器上的view需要做如下更改：
zone "shanks.com" {
         type    master;
         file    "shanks.com.zone";
         allow-transfer {
                 10.0.0.3;
         };
         notify  yes;
         also-notify {
                 10.0.0.3;
// 若未更改/var/named/chroot/etc/named.conf，可通过rndc reload加载配置文件，而不用重启
rndc reload
</pre>

**在主DNS上添加记录，若有从服务器，需要修改serial**
<pre>
// 在/var/named/chroot/etc/shanks.com.zone中，添加三条记录，并且修改serial
2001     ; serial
a           A            10.0.0.199
a           A            10.0.0.34
cname       CNAME        a.shanks.com.
// 通过rndc reload加载配置文件，并将配置同步变更给slave
rndc reload
// 添加MX记录，MX记录只能通过host查看
2002      ；serial
mx         MX  5         10.0.0.34
// rndc
rndc reload
// 查看记录
host mx.shanks.com 10.0.0.3
</pre>

**PTR解析**
<pre>
// 编辑master节点上的/var/named/chroot/etc/0.10.zone
$TTL 3600       ; 1 hour
@                  IN SOA  op.shanks.com. dns.shanks.com. (
                                 2004       ; serial
                                 900        ; refresh (15 minutes)
                                 600        ; retry (10 minutes)
                                 86400      ; expire (1 day)
                                 3600       ; minimum (1 hour)
                                 )   
                         NS      op.shanks.com.
34.0       IN     PTR           test.shanks.com.
199.0      IN     PTR           zabbix.shanks.com.
// 编辑master点上的/var/named/chroot/etc/view.conf
zone "0.10.in-addr.arpa" {
        type    master;
        file    "0.10.zone";
        allow-transfer {
                10.0.0.3;
        };  
        notify  yes;
        also-notify {
                10.0.0.3;
        };
  };
// slave上编辑/var/named/chroot/etc/view.conf，添加如下：
zone "0.10.in-addr.arpa" {
        type    slave;
        #',' as deliter
        masters {10.0.0.4;};
        file    "slave.0.10.zone";
  };
};
// 加载配置文件，并同步给slave
rndc reload
</pre>

**部署智能DNS**
<pre>
// 编辑master上的/etc/named.conf,在includes上面添加：
acl group1 {
  10.0.0.3;
};
acl group2 {
  10.0.0.4;
};
// 编辑master上的/var/named/chroot/etc/view.conf
view "GROUP1" {
  match-clients { group1; };
  zone "viewshanks.com" {
    type master;
    file "group1.viewshanks.com.zone";
        allow-transfer {
                10.0.0.3;
        };  
        notify  yes;
        also-notify {
                10.0.0.3;
        };  
  };  
};

view "GROUP2" {
  match-clients { group2; };
  zone "viewshanks.com" {
    type master;
    file "group2.viewshanks.com.zone";
        allow-transfer {
                10.0.0.3;
        };  
        notify  yes;
        also-notify {
                10.0.0.3;
        };  
  };  
};
// 编辑master上的/var/named/chroot/etc/group1.viewshanks.com.zone
$ORIGIN .
$TTL 3600       ; 1 hour
viewshanks.com                  IN SOA  op.viewshanks.com. dns.viewshanks.com. (
                                2005      ; serial
                                900        ; refresh (15 minutes)
                                600        ; retry (10 minutes)
                                86400      ; expire (1 day)
                                3600       ; minimum (1 hour)
                                )   
                        NS      op.viewshanks.com.
$ORIGIN viewshanks.com.
op          A            10.0.0.4
view        A            10.0.0.34
abc         A            10.0.0.34
// 编辑master上的/var/named/chroot/etc/group2.viewshanks.com.zone
$ORIGIN .
$TTL 3600       ; 1 hour
viewshanks.com                  IN SOA  op.viewshanks.com. dns.viewshanks.com. (
                                2005      ; serial
                                900        ; refresh (15 minutes)
                                600        ; retry (10 minutes)
                                86400      ; expire (1 day)
                                3600       ; minimum (1 hour)
                                )   
                        NS      op.viewshanks.com.
$ORIGIN viewshanks.com.
op          A            10.0.0.4
view        A            10.0.0.199
abc         A            10.0.0.199
// rndc reload
rndc reload
</pre>

**压测**
安装queryperf<br/>
<pre>
下载bind源码,wget http://ftp.isc.org/isc/bind9/9.7.3/bind-9.7.3.tar.gz
解压bind源码：tar zxf bind-9.7.3.tar.gz
进入解压后bind源码目录：cd bind-9.7.3/contrib/queryperf/
编译 ./configure
make
会在当前目录下出现queryperf，可以将它拷贝至/usr/bin/下
编辑域名记录文件{test.txt},格式如下：
    www.baidu.com  A
    www.baidu.com  A
    www.baidu.com  A
    www.baidu.com  A
压测命令：queryperf -f test.txt -s 8.8.8.8
</pre>
