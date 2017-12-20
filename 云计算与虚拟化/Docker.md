#Docker#
##Docker的安装使用##
docker基于内核虚拟化技术，遵循基础设施不可变原则。docker里面运行的程序只能以前台进程的方式运行。
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
2. 数据卷容器
  - --volumes-from<br/>
 