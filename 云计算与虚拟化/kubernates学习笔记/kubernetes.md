#kubernetes学习笔记（一）#

##一、基本概念##
###kubernetes是什么?###
Kubenetes是一款由Google开发的开源的容器编排工具，是一个全新的基于容器技术的分布式架构领先方案，是一个开放的开发平台，同时也是一个完备的分布式系统支撑平台。
###为什么要使用kubernetes？###
使用Kubernetes的理由很多，最根本的一个理由就是：IT从来都是一个由新技术驱动的行业。
使用Kubernetes所带来的好处：

    首先，最直接的感受就是我们可以“轻装上阵”地开发复杂系统了；
    其次，使用Kubernetes就是在全面拥抱微服务架构；
    然后，我们的系统可以随时随地整体“搬迁”到公有云上；
    最后，Kubernetes系统架构具备了超强的横向扩容能力。
###如何使用kubernetes？我们先看看kubernetes的相关术语###
####Master节点####
Kubernetes API Server（kube-apiserver):<br/>
提供HTTP Rest接口的关键服务进程，是kubernetes里所有资源的增、删、改、查等操作的唯一入口，也是集群控制的入口。<br/>
Kubernetes Controller Manager(kube-controller-manager):<br/>
kubernetes里所有资源对象的自动化控制中心，可以理解为资源对象的“大总管”。<br/>
Kubernetes Scheduler(kube-scheduler):<br/>
负责资源调度(pod调度)的进程，相当于公交车公司的“调度室”。

####Node节点####
kubelet:<br/>
负责pod对应的容器的创建、启停等任务，同时与Master节点密切协作，实现集群管理的基本功能。<br/>
kube-proxy:<br/>
实现kubernetes service的通信与负载均衡机制的重要组件。<br/>
Docker Engine(docker):<br/>
Docker引擎，负责本机的容器创建和管理工作。

##二、kubernetes的使用##
1.环境准备（服务器采用centos7 64位）
<pre>
//关闭防火墙
# systemctl disable firewalld
# systemctl stop firewalld
//安装etcd和kubernetes软件，安装时会自动安装docker软件
# yum install -y etcd kubernetes
//安装好软件后，修改两个配置文件：
//1.docker配置文件/etc/sysconfig/docker，其中options的内容设置为：
OPTIONS='--selinux-enabled=false --insecure-registry gcr.io'
//2.kubernetes apiserver配置文件为/etc/kubernetes/apiserver,把--admission_control参数中的ServiceAccount删除。
//按顺序启动所有的服务
# systemctl start etcd
# systemctl start docker
# systemctl start kube-apiserver
# systemctl start kube-controller-manager
# systemctl start kube-scheduler
# systemctl start kubelet
# systemctl start kube-proxy
</pre>





