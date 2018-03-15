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

##kubernetes一站式管理平台ku8 eye##
地址：https://github.com/bestcloud/ku8eye

Kubernetes安装选项：<br/>
本地开发环境:<br/>
Minikube安装(https://kubernetes.io/docs/getting-started-guides/minikube/)<br/>
Kubernetes集群：<br/>
1.Kubeadm(https://kubernetes.io/docs/setup/independent/install-kubeadm/）<br/>
2.使用IBM Clound Private Ce版本安装(https://hub.docker.com/r/ibmcom/cfc-installer/)<br/>
3.选择Bluemix,AWS,GCE,阿里，腾讯云等云的服务.<br/>

kubernetes参考学习：<br/>
About OCI：<br/>
[https://www.opencontainers.org](https://www.opencontainers.org)<br/>
runc on github:<br/>
[https://github.com/opencontainers/runc](https://github.com/opencontainers/runc)<br/>
OCI testing:<br/>
[https://github.com/opencontainers/runtime-tools](https://github.com/opencontainers/runtime-tools)<br/>
ContainerD:<br/>
[https://github.com/containerd/containerd](https://github.com/containerd/containerd)<br/>
CRI-O:<br/>
[http://cri-o.io](http://cri-o.io)<br/>
[https://github. com/kubernetes-incubator/cri-o](https://github.com/kubernetes-incubator/cri-o)

##kubernetes网络##
1.calico<br/>
2.NSX-t<br/>
3.flannel<br/>
外网访问：<br/>
1.nodePort<br/>
2.loadbalancer<br/>
3.Ingress

##kubernetes存储##
###K8s存储的主要应用场景###
1.应用程序/服务存储状态、数据提取等<br/>
2.应用程序/服务配置文件读取、秘钥配置等<br/>
3.不同应用程序间或者应用程序内进程间共享数据<br/>

##kubernetes日志和监控##
###日志的分类###
1.K8S的日志<br/>
2.K8S Cluster里面部署的应用程序的日志<br/>

分析处理:<br/>
常见的方案比如ElasticSearch + LogStash + Kibana的方案

容器当中的日志怎么收集:<br/>
1.让每个应用自行上传自己的日志<br/>
2.附加专用日志上传容器(side-car模式）<br/>
  1).在每一个pod中包含一个日志上传容器<br/>
  2).应用容器和日志容器通过共享卷交换日志数据<br/>
3.使用Docker引擎的日志收集功能<br/>
  1).利用Docker Log Driver收集每个容器的标准输出<br/>
  2).容器的标准输出会被写到宿主机的日志目录<br/>
  3).日志上传容器从宿主机的日志目录上传日志<br/>
  4).将整套日志收集系统从用户应用中分离出来<br/>

[https://docs.docker.com/engine/admin/logging/overview/#supported-logging-drivers](https://docs.docker.com/engine/admin/logging/overview/#supported-logging-drivers)

###kubernetes监控###
专用的容器级别的监控方案：cAdvisor/Heapster<br/>
常用的方案：<br/>
1.heapster + influxDB + Grafana(google有现成的部署实例)<br/>
2.heapster + prometheus + Grafana<br/>


##kubernetes应用部署##
###Helm架构###
1.Helm
2.Tiller
3.Chart package/repository

##kubernetes catlog##
catlog service

##kubernetes 实战##

某银行架构：
Terraform<br/>
Application Catalog、Orchestration helm、ingress controller<br/>
Service Mesh:spring cloud、istio<br/>
Log Analytics: elk<br/>
Metrics: prometheus、grafana、collectd、cAdvisor、Heapster、InfluxDB<br/>
Messaging: kafka<br/>
Security: keystone<br/>
Repos: harbor<br/>

PaaS云方案特点：<br/>
1.安装升级: 通过容器管理所有的PaaS云系统服务，便于安装，升级。<br/>
2.弹性伸缩：基于时间（Scheduler Resource)。<br/>
3.多集群管理：可以实现多集群应用部署和跨集群负载均衡。<br/>
4.DevOps：通过集成Jenkins实现了DevOps的功能，包括代码托管，镜像自动编译，应用自动部署，自动产生DevOps分析报告的功能。<br/>
5.微服务治理：通过istio实现了微服务治理的功能，主要包括分流，限流，熔断，服务发现，服务跟踪，服务拓扑结构展示等功能。<br/>
6.外部服务集成：通过service catlog实现了和外部应用对接的能力。例如ICP可以通过service catlog对接外部的一些数据库服务，第三方PaaS平台上的一些服务等等。<br/>
7.多云集成：通过集成Terraform实现了对不同云平台的集成。<br/>

