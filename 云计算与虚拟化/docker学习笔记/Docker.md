#Docker#
##Docker的安装使用##
docker基于内核虚拟化技术，遵循基础设施不可变原则。docker里面运行的程序只能以前台进程的方式运行。<br/>

Docker的安装可以指定docker源:<br/>
<pre>
$ cat /etc/yum.repos.d/docker.repo<<-EOF
[dockerrepo]
name=Docker Repository
baseurl=https://yum.dockerproject.org/repo/main/centos/7
enabled=1
gpgcheck=1
gpgkey=https://yum.dockerproject.org/gpg
EOF
</pre>

<pre>
// docker的安装
$ yum install -y docker
// docker的启动
$ systemctl start docker
// 查看docker镜像
$ docker images
// 搜索docker镜像
$ docker search centos
// pull centos
$ docker pull centos
// 保存镜像
$ docker save -o centos.tar centos
// 导入镜像
$ docker load --input centos.tar 或  docker load < centos.tar
// 删除镜像
$ docker rmi IMAGE ID
// 启动docker容器
$ docker run centos /bin/echo 'hello world'
// 查看所有进程，包括停止的和正在执行的
$ docker ps -a
// 带参数启动docker容器，退出用exit
$ docker run --name mydocker -t -i centos /bin/bash
// 启动之前开启过的docker
$ docker start mydocker
// 停止docker容器
$ docker stop mydocker
// 进入docker,docker提供，单用户模式，生产环境一般不用，生产环境用nsenter。
$ docker attach mydocker
// 安装提供nsenter命令的包，要是有则不用安装
$ yum install -y util-linux
// 获取docker容器PID
$ docker inspect -f "{{ .State.Pid}}" mydocker
// 通过获取的PID进入容器
$ nsenter -t 23447 -m -u -i -n -p

// 编写脚本完成登录docker
$ vim DockerIn.sh
#!/bin/bash
# Use nsenter to access docker

docker_in(){
    NAME_ID=$1
    PID=$(docker inspect -f "{{ .State.Pid}}" $NAME_ID)
    nsenter -t $PID -m -u -i -n -p
}

docker_in $1

// 不进入docker，执行某条命令
$ docker exec mydocker "echo"
// 通过exec也可以进入docker，exit退出时docker不会退出，但是可能会有莫名的问题
$ docker exec -it mydocker /bin/bash
// 删除容器,可以加 -f参数删除一个正在运行的docker
$ docker rm mydocker
// 启动容器执行完命令后删除
$ docker run --rm centos /bin/echo "hehe"
</pre>

docker使用优点：<br/>
1. simplifying Configuration
2. code Pipeline Management
3. Developer Productivity
4. app Isolation
5. Server Consolidation
6. Debugging Capabilities
7. Multi-tenancy
8. Rapid Deployment

面向产品：产品交付<br/>
面向开发：简化环境配置<br/>
面向测试：多版本测试<br/>
面向运维：环境一致性<br/>
面向架构：自动化扩容(微服务)<br/>

**docker网络**<br/>
端口映射：<br/>
1. 随机映射<br/>
   - docker run -P<br/>
2. 指定映射<br/>
   - -p hostPort:containerPort<br/>
   - -p ip:hostPort:containerPort<br/>
   - -p ip::containerPort<br/>
   - -p hostPort:containerPort:udp<br/>
   - -p 81:80 -p 443:443<br/>
<pre>
// 端口随机映射
$ docker run -P --name mynginx nginx
// 查看NAT情况
$ iptables -t nat -vnL
// 指定端口映射
$ docker run -p 80:80 --name mynginx nginx
</pre>

docker通过NAT映射模式访问网络，会影响性能。<br/>

**docker数据管理**<br/>
1. 数据卷<br/>
  - -v /data<br/>
  - -v src:dst<br/>
栗子：<br/>
<pre>
// 通过-v /data方式挂载
$ docker run -d --name nginx-volume-test1 -v /data nginx
// 查看挂载点
$ docker inspect -f '{{ .Mounts}}' nginx-volume-test1
// 通过-v /data/nginx-volume-test2方式挂载，/data后面加:ro,以只读方式挂载
$ docker run -d --name nginx-volume-test2 -v /data/nginx-volume-test2:/data nginx
// 可以挂载单个文件进去
$ docker run --rm -it -v /root/.bash_history:/.bash_history nginx /bin/bash
</pre>
2. 数据卷容器（数据在多个容器间共享）<br/>
  - --volumes-from<br/>
栗子：<br/>
<pre>
// 即使nginx-volume-test2停了，依旧可以访问,将多个docker容器的数据交流联系起来了
$ docker run -it --name volume-test3 --volumes-from nginx-volume-test2 centos /bin/bash
</pre>
 

###docker网络###
--基于link的互联--
<pre>
//容器默认支持互相联通，使用--icc=false隔离容器
docker run --rm=true --link=mysqlserver:myserver -it java /bin/bash
</pre>

--直接使用宿主机的网络--
<pre>
docker run --rm=true --net=host --name=mysqlserver -e MYSQL_ROOT_PASSWORD=123456 mysql
</pre>
--容器共用一个IP网络--
<pre>
docker run --rm=true --net=container:mysqlserver java ip addr
</pre>
##镜像构建##
干掉所有正在运行的容器：<br/>
<pre>
// 杀掉所有容器
$ docker kill $(docker ps -a -q)
// 删除所有容器
$ docker rm $(docker ps -a -q)
</pre>

**镜像构建的两种方式**：<br/>
1. 手动构建<br/>
<pre>
// 启动基础镜像
$ docker run --name mynginx -it centos
// 在启动镜像后的容器内安装nginx
$ curl -o /etc/yum.repos.d/epel.repo http://mirrors.aliyun.com/repo/epel-7.repo
$ yum install -y nginx && yum clean all
$ echo "daemon off;" >> /etc/nginx/nginx.conf
// 退出镜像后，提交创建的docker镜像
$ docker commit -m 'My Nginx' 86da5d687406 oldboy/mynginx:v1
// 启动容器
$ docker run -d --name mynginx-v1 -p 80:80 oldboy/mynginx:v1 nginx
</pre>
2. Dockerfile构建<br/>
<pre>
// 创建构建镜像的目录
mkdir -p /opt/dockerfile/nginx
// 创建Dockerfile文件,文件名为Dockerfile，且D必须大写，内容如下：
# This Dockerfile for nginx

#Base image
FROM centos

#MAINTAINER
MAINTAINER Wang Fei 429390237@qq.com

#Commands
RUN curl -o /etc/yum.repos.d/epel.repo http://mirrors.aliyun.com/repo/epel-7.repo
RUN yum install -y nginx && yum clean all
RUN echo "daemon off;" >> /etc/nginx/nginx.conf
ADD index.html /usr/share/nginx/html/index.html
EXPOSE 80
CMD ["nginx"]

// 创建index.html
$ echo "nginx in docker,hahaha" > index.html
// 构建镜像
$ docker build -t mynginx:v2 .
// 启动镜像
$ docker run --name mynginx-v2 -d -p 81:80 mynginx:v2
</pre>

Dockerfile主要配置：<br/>
- FROM （指定基础镜像）<br/>
- MAINTAINER (指定维护者信息) <br/>
- RUN (在要运行的命令前面加上RUN即可) <br/>
- ADD （copy文件，会自动解压）<br/>
- WORKDIR （设置当前工作目录） <br/>
- VOLUME (设置卷，挂载主机目录) <br/>
- EXPOSE (指定对外的端口) <br/>
- CMD （指定容器启动后要干的事情） <br/>
- ENTRYPOINT （指定容器启动后要干的事情，不可被覆盖）
Dockerfile参考：
[https://docs.docker.com/engine/reference/builder/](https://docs.docker.com/engine/reference/builder/ "Dockerfile reference")

Docker Registry私有仓库 Nginx+认证的方式：</br>
1. 申请免费的SSL证书: [https://buy.wosign.com/free/](https://buy.wosign.com/free/)<br/>
2. 参照部署指南进行证书部署<br/>
3. 设置验证<br/>

==============================harbor仓库=====================================<br/>
http://vmware.github.io/harbor/index_cn.html<br/>
https://github.com/vmware/harbor<br/>