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