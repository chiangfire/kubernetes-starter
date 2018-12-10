## Docker基础
### 1，安装Docker
```bash
yum -y update                                         --更新 yum 源到最新

cat >/etc/yum.repos.d/docker.repo <<-EOF              --添加 yum 源 如下
    [dockerrepo]
	name=Docker Repository
	baseurl=https://yum.dockerproject.org/repo/main/centos/7
	enabled=1
	gpgcheck=1
	gpgkey=https://yum.dockerproject.org/gpg
EOF

yum install -y docker-selinux                         --安装 docker-selinux

yum install -y docker-engine                          --安装 docker-engine

systemctl start docker.service                        --立即启动 docker 服务

systemctl enable docker.service                       --设置 docker 开机服务启动
```
### 2，Docker配置和使用
```bash
docker info                                           --查看 docker 相关信息

docker version                                        --查看 docker 服务端和客户端相关版本

docker search '镜像名称'                              --查询 docker 镜像

docker pull '镜像名称'                                --下载或更新 docker 镜像 <docker pull '镜像名称':'版本'>

docker images                                        --查看本地已经装载<下载>的镜像

docker run -it java java -version                    --docker run:执行容器，-it:打开命令终端，java:执行名叫java的镜像，java -version:打开命令终端后执行该命令

docker run java env                                  --查看名叫 java 的 docker 镜像的环境变量

docker run 后面追加-d=true或者-d                      --容器将会运行在后台模式。

docker run 后面追加 -rm                               --容器运行完成后立即删除

docker exec                                           --进入到到该容器中，或者attach重新连接容器的会话 <不建议使用 attach，一个是会话的原因，一个是断开后会停止 docker 镜像>

docker create/start/stop/pause/unpause                --容器生命周期相关指令

    docker create -it --name=myjava java java -version    --创建一个 docker 容器名叫 myjava，后面的命令就是容器所要做的事情
	
	docker start myjava                                   --启动刚刚创建的容器<测试>
	
	docker create --name=mysql -e MYSQL_ROOT_PASSWORD=jiang -p 3306:3306 mysql   --建一个 mysql 的 docker 容器，-e是往容器环境变量里面设值<一般服务的配置也是通过这个传递的>，-p 3306:3306 是将主机的3306转发到容器的3306端口《可使用--net=host替代直接使用宿主机端口不做转发》，最后的mysql是镜像的名称
	
	docker start '容器名称 || 容器ID'                     --启动刚刚创建的 mysql docker容器
	
	docker exec -it '容器名称 || 容器ID' bash             --进入容器 <如果报没有 bash 的错误直接使用：docker exec -it '容器名称' sh>进入容器
	
	docker stop '容器名称'                            --停止 mysql docker容器

docker ps -a                                          --查看 docker 所有的容器

docker rm 'id'                                        --删除容器  'id' 就是 docker ps -a 查看到的id

docker ps                                             --查看 dicker 当前正在运行的容器

docker logs -f '容器ID'                               --查看 服务容器运行日志 <就是用docker ps命令显示出来的那个容器ID>

docker commit 'id' 镜像名称                           --将容器创建为镜像 'id'=容器ID《docker ps -a 查看到的id》，《所有镜像可使用 docker images 命令查看》

/usr/lib/systemd/system/docker.service                --docker 配置文件地址

tail -f /var/log/messages |grep docker                --查看 docker 日志，这个日志目录应该是不对的

docker login                                          --docker 登陆，然后提示 输入用户名，密码

docker tag zookeeper:3.5 test/zookeeper:3.5           --为镜像 zookeeper 打上tag <:3.5是镜像版本>

git push test/zookeeper:3.5                           --将zookeeper 镜像上传到仓库<:3.5是镜像版本>
```
### 3，Dockerfile使用
```bash
touch Dockerfile                                      --创建Dockerfile文件
vi Dockerfile                                         --编辑文件
    FORM centos                                       --'FORM' 在某个基础镜像之上进行扩展<centos:latest；:latest是版本>
	MAINTAINER chiangfire@outlook.com                 --'MAINTAINER' 镜像创建者
	ADD nginx-1.12.2.tar.gz /usr/local/src            --'ADD' 添加 nginx-1.12.2.tar.gz 文件到 /usr/local/src
	EXPOSE 6379                                       --'EXPOSE' 镜像开放6379端口
	ENTRYPOINT java -version                          --'ENTRYPOINT' 镜像启动时要执行的命令，必须是前台执行的方式，一般都是自己的应用系统启动命令
	ENV JAVA_HOME /usr/lib/java-8                     --'ENV' 添加环境变量
	RUN '创建镜像要执行的命令《比如安装软件》'        --多条命令可使用 \ 换行，下一行使用 && 开头，整个Dockerfile最好只有一个RUN因为每个RUN都是一层镜像
docker build -t '镜像名称' '镜像完成后所在目录'       --创建镜像，镜像完成后所在目录可使用 . 代表当前目录

Supervisor docker                                     --可存储密码，以及同时使用多个进程以及开启后台进程 <具体可 百度>
```
#### 4，Volume存储使用
```bash
docker run --rm=true -it -v /storage /leader javad /bin/bash                          --将本机目录 /storage 挂载到 javad 容器的 /bin/bash目录 <注意：容器删除目录还在>
docker run --rm=true --privileged=true -it -v /storage /leader javad /bin/bash        --和上面的命令一样只是加了 --privileged=true 可将多个镜像挂载到同一个目录，已达到文件共享
docker run --rm=true --privileged=true --volumes-from='容器ID' -it javad /bin/bash    --和上面的命令一样只是加了 --volumes-from='容器ID'，将某个容器挂载到 javad 容器的 /bin/bash目录
docker run --rm=true --link=127.0.0.1:myserver -it javad /bin/bash                    --使用--link=127.0.0.1:myserver，将远程容器挂载到 javad 容器的 /bin/bash目录《myserver是远程容器在当前容器的一个别名，可使用 ping myserver 查看可ping通》
docker run --rm=true --net=container:mysqlserver javad ip addr                        --容器共享同一个网络《mysqlserver=容器名称》，可用于多个服务使用同一个网络
docker inspect '容器ID'                                                               --查看文件所写的真实目录 《可直接在看到的目录下写数据》
```
### 5，Docker 路由机制打通网络《比较高效推荐使用》	《现在两个docker镜像128,130》
```bash
修改128镜像：
    vi /usr/lib/systemd/system/docker.service
	ExecStart=/usr/bin/docker daemon --bip=172.18.42.1/16 -H fd:// -H=unix:///var/run/docker.sock
	systemctl daemon-reload
	重启128 docker 镜像
	
128镜像上执行 route add -net 172.17.0.0/16 gw 192.168.18.130
130镜像上执行 route add -net 172.18.0.0/16 gw 192.168.18.128

可以 ping 看看能不能通，如果不同可清理防火墙规则试试


注：有时间看看 docker + open vSwitch 打通网络
```
### 6，Docker-Compose<半个容器编排，还是用 k8s 吧> 可以在容器中直接使用 service 名称 代替 IP，相互访问容器里面的 service；使用如下
```bash
1，定义 docker-compose.yml 内容如下：
    version: '3'                                     -- docker compose 版本
	service:                                         -- service 定义
	  message-service:                               -- service 名称
	    image: message-service:latest                -- service 镜像名称
		ports: 
		- 8080:8080                                  -- service 对外提供的端口
      
      user-service:                            -- service 名称
        image: user-service:latest             -- service 镜像
        command                                -- 命令行参数
        - "--mysql.address=192.168.0.1"		    -- 参数 就是 user-service 容器在启动时所添加的命令行参数，在spring配置文件里面可使用${mysql.address}	取到
		
	  user-edge-service:                       -- service 名称             
	      image: user-edge-service: latest     -- 镜像名称
		  links:                                -- 依赖 service
		  - user-servive                        -- 依赖 service 名称
		  - message-service                     -- 依赖 service 名称
		  command:                              -- 命令行参数
		  - "--redis.address=127.0.0.1"         -- 参数 就是 user-service 容器在启动时所添加的命令行参数，在spring配置文件里面可使用${redis.address}	取到
		  
		  
2，docker-compose up -d                        -- 后台运行 docker-compose
```
### 7，附录
```bash
netstat -nlpt                                         --查看所有端口映射情况
netstat -nlpt |grep 3306                              --查看3306端口使用情况
service mysqld stop                                   --停止名叫 mysqld 的服务 
mysql -uroot -p                                       --centos7 使用mysql
env                                                   --centos7 查看环境变量
```
### 8，删除Docker
```bash
yum list installed | grep docker                      --列出 docker 安装的软件包
yum -y remove '安装的软件包名'                        --卸载 docker 
rm -rf /var/lib/docker                                --删除 docker 镜像、容器，卷组和用户自配置文件。
```
### 9，说明
```bash
docker 默认支持互通，可通过 -icc=false 关闭互通。《/usr/bin/docker daemon --icc=false》
私有库搭建可使用：https://github.com/goharbor/harbor/releases	
```

 ![image](https://github.com/chiangfire/kubernetes-starter/blob/master/images/k8s-concept.jpg)
  **以下记录kubernetes在绿色网络环境下的集群搭建及集群的使用、常用命令、应用的部署。首先剥离了认证授权和服务发现模块，从最核心的模块开始构建集群，然后逐步增加认证授权和服务发现部分，在搭建过程中逐步熟悉kubernetes。**
  
## [一、环境准备][1-1] [centos][1-1] [ubuntu][1-2]
## [二、基础集群部署 - kubernetes-simple][2]
## [三、完整集群部署 - kubernetes-with-ca][3]
## [四、在kubernetes上部署我们的微服务][4]








  [1-1]: https://github.com/chiangfire/kubernetes-starter/blob/master/docs/1-pre-centos.md
  [1-2]: https://github.com/chiangfire/kubernetes-starter/blob/master/docs/1-pre-ubuntu.md
  [2]: https://github.com/chiangfire/kubernetes-starter/blob/master/docs/2-kubernetes-simple.md
  [3]: https://github.com/chiangfire/kubernetes-starter/blob/master/docs/3-kubernetes-with-ca.md
  [4]: https://github.com/chiangfire/kubernetes-starter/blob/master/docs/4-microservice-deploy.md
