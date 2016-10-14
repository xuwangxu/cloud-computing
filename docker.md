# Docker
## 1.什么是LXC？
LXC为Linux Container的简写。Linux Container容器是一种内核虚拟化技术，可以提供轻量级的虚拟化，以便隔离进程和资源，而且不需要提供指令解释机制以及全虚拟化的其他复杂性。相当于C++中的NameSpace。容器有效地将由单个操作系统管理的资源划分到孤立的组中，以更好地在孤立的组之间平衡有冲突的资源使用需求。与传统虚拟化技术相比，它的优势在于：

- 与宿主机使用同一个内核，性能损耗小；
- 不需要指令级模拟；
- 不需要即时(Just-in-time)编译；
- 容器可以在CPU核心的本地运行指令，不需要任何专门的解释机制；
- 避免了准虚拟化和系统调用替换中的复杂性；
- 轻量级隔离，在隔离的同时还提供共享机制，以实现容器与宿主机的资源共享。
## 2.docker 介绍
Docker 是Docker.lnc公司开源的一个基于LXC技术之上构建的contailer容器引擎，源代码托管在github上，基于go语言并遵从Apache2.0协议开源。
Docker是通过内核虚拟化技术（namespace及cgroups等）来提供容器的资源隔离与安全保障等，由于Docker通过操作系统层的虚拟化实现隔离，所以Docker容器在运行时，不需要类似虚拟机VM额外的操作系统开销，提高资源利用率。

Docker的设计目标：开发和运维可以在任何地方进行构建、传输、运行任何程序。

## 3.docker 的工作模式
Docker对于使用者是一个C/S的模式架构，而Docker的后端是一个松耦合的架构，模块各司其职，并有机会组合，支撑docker的运行。

使用者通过docker client与docker daemon建立通信，并发送请求给后者

docker daemon作为docker架构的主体部分，首先提供server的功能可以接受docker client请求；而后engine执行docker内部的一系列工作，每一项工作都是以一个JOB的存在。

job的运行过程当中，需要容器镜像的时候，则从Docker Registry中下载镜像，并通过镜像管理驱动graphdriver将下载镜像以Graph的形式存储；当需要为Docker创建网络环境时，通过网络管理驱动networkdriver创建并配置Docker容器网络环境；当需要限制Docker容器运行资源或执行用户指令等操作时，则通过execdriver来完成。而libcontainer是一项独立的容器管理包，networkdriver以及execdriver都是通过libcontainer来实现具体对容器进行的操作。当执行完运行容器的命令后，一个实际的Docker容器就处于运行状态，该容器拥有独立的文件系统，独立并且安全的运行环境等。

## 4.docker 的三大核心概念
###4.1 镜像（image）
- docker镜像就是一个只读的模板

    例如：一个镜像可以包含一个完整的CentOS操作系统，里面仅安装了Apache或其他用户需要的其他应用程序。

- 镜像可以用来创建docker容器
- Docker提供了一个很简单的机制来创建镜像或者更新现有的镜像，用户甚至可以直接从其他人那里下载一个已经做好的镜像来直接使用。
###4.2 容器（container）
- docker利用容器来运行应用
- 容器从创建的镜像来运行实例，它可以启动、开始、停止、删除。每个容器都是互相隔离的，保证安全的平台。
- 可以把容器看做一个简单的linux环境（root用户权限、进程空间、用户空间、网络空间等等）和运行在其中的程序
###4.3 仓库（repository）
- 仓库是集中存放镜像文件的场所。有时候把仓库和仓库注册服务器（Registry）混为一谈，并不严格区分。实际上，仓库注册服务器上往往存放着多个仓库，每个仓库中又包含了多个镜像，每个镜像有不同的标签(tag)。
- 仓库分为公开仓库(Public)和私有仓库(Private)两种形式。
- 最大的公开仓库是Docker Hub，存放了数量庞大的镜像供用户下载。国内的公开仓库包括Docker Pool等，可以提供大陆用户更稳定快读的访问。
- 当用户创建了自己的镜像之后就可以使用push命令将它上传到公有或者私有仓库，这样下载在另外一台机器上使用这个镜像时候，只需需要从仓库上pull下来就可以了。

## 5.docker快速入门
0.操作系统环境

	[root@linux-node1 ~]# cat /etc/redhat-release 
	CentOS Linux release 7.2.1511 (Core) 
	[root@linux-node1 ~]# uname -r
	3.10.0-327.18.2.el7.x86_64
1.启动docker

	[root@linux-node1 ~]# systemctl  start docker  
	[root@linux-node1 ~]# ifconfig  docker0   #<=启动docker的时候，创建了一个docker0的网桥
	docker0: flags=4099<UP,BROADCAST,MULTICAST>  mtu 1500
        inet 172.17.0.1  netmask 255.255.0.0  broadcast 0.0.0.0
        inet6 fe80::42:55ff:fea9:a09a  prefixlen 64  scopeid 0x20<link>
        ether 02:42:55:a9:a0:9a  txqueuelen 0  (Ethernet)
        RX packets 49  bytes 3709 (3.6 KiB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 40  bytes 10359 (10.1 KiB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

	[root@linux-node1 docker]# docker load  --input  centos.tar.gz  #<=导入镜像
	[root@linux-node1 docker]# docker images
	REPOSITORY          TAG                 IMAGE ID            CREATED             	SIZE
	docker.io/centos    latest              980e0e4c79ec        5 weeks ago         196.7 MB
	[root@linux-node1 docker]# docker rmi $(docker images| awk '{print $3}'|sed -n 2p)  #<=删除镜像命令 但是如果已经创建了一个容器，镜像不会被删除

    [root@linux-node1 docker]# docker run centos /bin/echo "look at me,bitch"
    look at me,bitch
    [root@linux-node1 docker]# docker ps 
    CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES


























