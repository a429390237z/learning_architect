##准备##
1.软件包选择：Basic<br/>
2.关闭iptables和SELinux<br/>
3.预装glusterfs软件包 <br/>
<pre>
# vim Centos-Base.repo
[storage]
name=CentOS-$releasever - storage - mirrors.aliyun.com
failovermethod=priority
baseurl=http://mirrors.aliyun.com/centos/$releasever/storage/$basearch/gluster-3.9/
        http://mirrors.aliyuncs.com/centos/$releasever/storage/$basearch/gluster-3.9/
#mirrorlist=http://mirrorlist.centos.org/?release=$releasever&arch=$basearch&repo=contrib
gpgcheck=0
enabled=1
# yum -y install glusterfs-server glusterfs-cli glusterfs-geo-replication
</pre>

##GlusterFS理论基础##
理论和实践分析，GlusterFS目前主要适用大文件存储场景，对于小文件尤其是海量小文件，存储效率和访问性能都变现不佳，海量小文件LOFS问题是工业界和学术界公认的难题，GlusterFS作为通用的分布式文件系统，并没有对小文件作额外的优化措施，性能不好也是可以理解的。<br/>
Media:文档、图片、音频、视频<br/>
Shared storage:云存储、虚拟化存储、HPC(高性能计算)<br/>
Big data:日志文件、RFID(射频识别)数据<br/>

##安装##
1.前期准备,同上</br>
2.修改主机名(/etc/sysconfig/network),使用host命令临时生效，修改配置文件永久生效。配置主机名关联(/etc/hosts)。<br/>
3.启动glusterfs<br/>
<pre>
# service glusterd start
# service glusterd status
# chkconfig glusterd on
</pre>
4.存储主机加入信任存储池<br/>
<pre>
# gluster peer probe node2
# gluster peer probe node3
# gluster peer probe node4
//查看状态
# gluster peer status
</pre>
5.安装xfs文件系统支持<br/>
<pre>
# yum install -y xfsprogs
</pre>
6.对磁盘进行分区，格式化<br/>
<pre>
//可以用fdisk和parted对磁盘进行分区，也可不做分区
# fdisk /dev/sdb
//格式化
# mkfs.xfs -f /dev/sdb
</pre>
7.创建挂载目录并挂载<br/>
<pre>
# mkdir -p /storage/brick1
# mount /dev/sdb /storage/brick1
</pre>
8.配置开机自动挂载<br/>
<pre>
# echo '/dev/sdb                /storage/brick1         xfs     defaults        0 0' >> /etc/fstab
</pre>
9.创建分布卷并启动<br/>
<pre>
//创建卷
# gluster volume create gv1 node1:/storage/brick1 node2:/storage/brick1 force
//启动卷
# gluster volume start gv1
//查看
# gluster volume info
</pre>
10.挂载<br/>
<pre>
# mount -t glusterfs 127.0.0.1:/gv1 /mnt
//用NFS方式挂载，需要确保卷是否禁止了nfs挂载
//查看卷是否禁止了nfs挂载，若禁止，需要启用，并重启卷，然后再挂载
# gluster volume info gv1
# gluster volume set gv1 nfs.disable off
# gluster volume stop gv1
# gluster volume start gv1
# mount -o mountproto=tcp -t nfs 192.168.0.4:/gv1 /mnt/
</pre>
10.创建复制卷并挂载<br/>
<pre>
//注意，若要使用nfs挂载方式，开启方法同上
# gluster volume create gv2 replica 2 node3:/storage/brick1 node4:/storage/brick1 force
# gluster volume start gv2
# mount -t glusterfs node1:/gv2 /opt
</pre>
11.创建条带卷并挂载<br/>
<pre>
# gluster volume create gv3 stripe 2 node3:/storage/brick2 node4:/storage/brick2 force
# gluster volume start gv3
# mount -t glusterfs node1:/gv3 /data
</pre>

注意：挂载点目录最好单独创建，不要使用系统已有的目录<br/>

12.添加磁盘到复制卷<br/>
<pre>
//首先停止卷，然后添加，再启动
# gluster volume stop gv2
# gluster volume add-brick gv2 replicas 2 node1:/storage/brick1 node2:/storage/brick1 force
</pre>
13.添加磁盘后，需要做磁盘平衡<br/>
<pre>
# gluster volume rebalance gv2 start
</pre>
##部分概念##
Distributed:分布式卷，文件通过hash算法随机的分布到有bricks组成的卷上。<br/>
Replicated：复制式卷，类似raid1，replica数必须等于volume中brick所包含的存储服务器数，可用性高。<br/>
Striped: 条带式卷，类似于raid0，stripe数必须等于volume中brick所包含的存储服务器数。<br/>
Distributed Striped:分布式的条带卷，volume中brick所包含的存储服务器数必须是stripe的倍数(>=2倍),兼顾分布式和条带式的功能.<br/>
Distributed Replicated:分布式的复制卷，volume中brick所包含的存储服务器数必须是replica的倍数(>=2倍),兼顾分布式和复制式的功能。<br/>

