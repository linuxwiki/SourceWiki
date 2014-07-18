# 轻量级虚拟化Docker

## Docker基本介绍

Docker发端于一个名为dotcloud的开源项目；随着编写者不断挖掘它的潜力，它迅速变成了一个炙手可热的项目。它由GO语言编写的，并且只支持Linux。它基于Linux容器（LxC）来创建一个虚拟环境。Docker不会通过建立独有的操作系统、进程和对硬件进行模拟来创建属于自己的虚拟机。请注意：虚拟环境VE(Virtual Environment)和虚拟机（VM）很不一样。虚拟机是由虚拟工具或者模拟器（HyperV 、VMWare等）创建的，是一个全镜像的主机源，其中包括操作系统、硬盘调整、网络和虚拟进程。过于臃肿的结构吃掉了大量的硬盘空间同时拖慢了运行和开机速度。
 
一台VE就像是轻量级的VM，它在已有的内核关于底层硬件的镜像上建立一个可以用来运行应用的‘容器’。它也可以用来创建操作系统，因为所谓的操作系统也不过是一个跑在内核上的应用而已。可以把Docker想象成LxC的一个强化版，只是具有以下LxC所不具有的特性：
 
* **强大的可移植性**：你可以使用Docker创造一个绑定了你所有你所需要的应用的对象。这个对象可以被转移并被安装在任何一个安装了 Docker 的 Linux 主机上。
* **版本控制**： Docker自带git功能，能够跟踪一个容器的成功版本并记录下来，并且可以对不同的版本进行检测，提交新版本，回滚到任意的一个版本等功能等等。
* **组件的可重用性**： Docker 允许创建或是套用一个已经存在的包。举个例子，如果你有许多台机器都需要安装 Apache 和 MySQL 数据库，你可以创建一个包含了这两个组件的‘基础镜像’。然后在创建新机器的时候使用这个镜像进行安装就行了。
* **可分享的类库**：已经有上千个可用的容器被上传并被分享到一个共有仓库中[http://index.docker.io/](http://index.docker.io/)。考虑到AWS对于不同环境下的调试和发布，这一做法是十分聪明的。
 
> LxC是一个Linux提供的收容功能接口，通过LxC提供的API和简单的工具，使得Linux用户可以简单的创建和管理系统或者应用的空间。[LXC容器](http://www.ibm.com/developerworks/cn/linux/l-lxc-containers/)
 
Docker通常用于如下场景：
 
    web应用的自动化打包和发布；
    自动化测试和持续集成、发布；
    在服务型环境中部署和调整数据库或其他的后台应用；
    从头编译或者扩展现有的OpenShift或Cloud Foundry平台来搭建自己的PaaS环境。
 
Docker并不是全能的，设计之初也不是KVM之类虚拟化手段的替代品：
 
    Docker是基于Linux 64bit的，无法在windows/unix或32bit的linux环境下使用
    LXC是基于cgroup等linux kernel功能的，因此container的guest系统只能是linux base的
    隔离性相比KVM之类的虚拟化方案还是有些欠缺，所有container公用一部分的运行库
    网络管理相对简单，主要是基于namespace隔离
    cgroup的cpu和cpuset提供的cpu功能相比KVM的等虚拟化方案相比难以度量
    docker对disk的管理比较有限
    container随着用户进程的停止而销毁，container中的log等用户数据不便收集
 
Docker实践解决方案：
 
    隔离性：Docker在文件系统和网络级别隔离了应用。从这个意义上来讲很像在运行”真正的“虚拟机。
    重复性：用你喜欢的方式准备系统（登录并在所有软件里执行apt-get命令，或者使用Dockerfile），然后把修改提交到镜像中。你可以随意实例化若干个实例，或者把镜像传输到另一台机器，完全重现同样的设置。
    安全性：Docker容器比普通的进程隔离更为安全。Docker团队已经确定了一些安全问题，正在着手解决。
    资源约束：Docker现在能限制CPU的使用率和内存用量。目前还不能直接限制磁盘的使用情况。
    易于安装：Docker有一个Docker Index，这个仓库存储了现成的Docker镜像，你用一条命令就可以完成实例化。比如说，要使用Clojure REPL镜像，只要运行docker run -t -i zefhemel/clojure-repl命令就能自动获取并运行该镜像。
    易于移除：不需要应用了？销毁容器就行。
    升级、降级：和EC2VM一样：先启动应用的新版本，然后把负载均衡器切换到新的端口。
    快照、备份：Docker能提交镜像并给镜像打标签，和EC2上的快照不同，Docker是立即处理的。
 
 
* [Docker Getting Start: Related Knowledge ](http://tiewei.github.io/cloud/Docker-Getting-Start/)
* [谁是容器中的“战斗机”？Docker与Chef、LXC等容器对比](http://code.csdn.net/news/2819773)
* [Docker：利用Linux容器实现可移植的应用部署](http://www.infoq.com/cn/articles/docker-containers)

## docker 安装配置

### Docker install
 
Docker的安装非常简单，这里只介绍Ubuntu 14.04的安装，其他发行版本的安装可以参考官网手册。
 
``` bash
$ sudo apt-get update
$ sudo apt-get install docker.io
$ sudo ln -sf /usr/bin/docker.io /usr/local/bin/docker
```
 
获取当前docker版本
 
``` bash
$ sudo docker version
Client version: 0.9.1
Go version (client): go1.2.1
Git commit (client): 3600720
Server version: 0.9.1
Git commit (server): 3600720
Go version (server): go1.2.1
Last stable version: 0.11.1, please update docker
```
 
* [Docker install on ubuntu](http://docs.docker.io/installation/ubuntulinux/)
 
## Docker images
 
* [Docker index](https://index.docker.io/) Docker镜像首页，包括官方镜像和其它公开镜像
 
### Search index images
 
``` bash
$ sudo docker search ubuntu
```
 
### Pull images
 
``` bash
$ sudo docker pull ubuntu # remote index 获取ubuntu官方镜像
$ sudo docker images # 查看当前镜像列表
REPOSITORY          TAG                 IMAGE ID            CREATED             VIRTUAL SIZE
ubuntu              13.10               5e019ab7bf6d        3 weeks ago         180 MB
ubuntu              saucy               5e019ab7bf6d        3 weeks ago         180 MB
ubuntu              12.04               74fe38d11401        3 weeks ago         209.6 MB
... ...
```
 
### Running an interactive shell
 
``` bash
$ sudo docker run -i -t ubuntu:14.04 /bin/bash
```

* docker run - 运行一个容器
* -t - 分配一个（伪） tty (link is external)
* -i - 开发输入(so we can interact with it)
* ubuntu - 使用ubuntu基础镜像
* /bin/bash - 运行bash shell

 
* ubuntu会有多个版本，通过指定tag来启动特定的版本[image]:[tag]
 
``` bash
$ sudo docker ps # 查看当前运行的容器, ps -a列出当前系统所有的容器
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
6c9129e9df10        ubuntu:14.04        /bin/bash           6 minutes ago       Up 6 minutes                            cranky_babbage
```

### 相关快捷键

* 退出：`Ctrl-D` or `exit`
* detach：`Ctrl-p + Ctrl-q`
* attach: `docker attach CONTAINER ID`


## 配置网络

Docker uses Linux bridge capabilities to provide network connectivity to containers. The docker0 bridge interface is managed by Docker for this purpose. When the Docker daemon starts it :
 
Dokcer通过使用Linux桥接提供容器之间的通信，docker0桥接接口的目的就是方便Docker管理。当Docker daemon启动时需要做以下操作：
 
* creates the docker0 bridge if not present 如果docker0不存在则创建
* searches for an IP address range which doesn’t overlap with an existing route 搜索一个与当前路由不冲突的ip段
* picks an IP in the selected range 在确定的范围中选择ip
* assigns this IP to the docker0 bridge 绑定ip到docker0
 
### 列出当前主机网桥
 
``` bash
$ sudo brctl show  # brctl工具依赖bridge-utils软件包
bridge name bridge id STP enabled interfaces
docker0 8000.000000000000 no
```
 
### 查看当前docker0 ip
 
``` bash
$ sudo ifconfig docker0
docker0 Link encap:Ethernet HWaddr xx:xx:xx:xx:xx:xx
inet addr:172.17.42.1 Bcast:0.0.0.0 Mask:255.255.0.0
```
 
在容器运行时，每个容器都会分配一个特定的虚拟机口并桥接到docker0。每个容器都会配置同docker0 ip相同网段的专用ip地址，docker0 的IP地址被用于所有容器的默认网关。
 
### 运行一个容器
 
``` bash
$ sudo docker run -t -i -d ubuntu /bin/bash
52f811c5d3d69edddefc75aff5a4525fc8ba8bcfa1818132f9dc7d4f7c7e78b4
$ sudo brctl show
bridge name bridge id STP enabled interfaces
docker0 8000.fef213db5a66 no vethQCDY1N
```
 
以上, docker0 扮演着52f811c5d3d6 container这个容器的虚拟接口vethQCDY1N interface桥接的角色。
 
#### 使用特定范围的IP
 
Docker会尝试寻找没有被主机使用的ip段，尽管它适用于大多数情况下，但是它不是万能的，有时候我们还是需要对ip进一步的规划。Docker允许你管理docker0桥接或者通过`-b`选项自定义桥接网卡，需要安装`bridge-utils`软件包。
 
基本步骤如下：
 
* ensure Docker is stopped  确保docker的进程是停止的
* create your own bridge (bridge0 for example) 创建自定义网桥
* assign a specific IP to this bridge   给网桥分配特定的ip
* start Docker with the -b=bridge0 parameter    以-b的方式指定网桥
 
``` bash
# Stop Docker
$ sudo service docker stop
# Clean docker0 bridge and
# add your very own bridge0
$ sudo ifconfig docker0 down
$ sudo brctl addbr bridge0
$ sudo ifconfig bridge0 192.168.227.1 netmask 255.255.255.0
# Edit your Docker startup file [ubuntu12.04配置文件为docker.io]
$ echo "DOCKER_OPTS=\"-b=bridge0\"" >> /etc/default/docker
# Start Docker
$ sudo service docker start
# Ensure bridge0 IP is not changed by Docker
$ sudo ifconfig bridge0
bridge0 Link encap:Ethernet HWaddr xx:xx:xx:xx:xx:xx
inet addr:192.168.227.1 Bcast:192.168.227.255 Mask:255.255.255.0
# Run a container
$ docker run -i -t ubuntu /bin/bash
# Container IP in the 192.168.227/24 range
root@261c272cd7d5:/# ifconfig eth0
eth0 Link encap:Ethernet HWaddr xx:xx:xx:xx:xx:xx
inet addr:192.168.227.5 Bcast:192.168.227.255 Mask:255.255.255.0
# bridge0 IP as the default gateway
root@261c272cd7d5:/# route -n
Kernel IP routing table
Destination Gateway Genmask Flags Metric Ref Use Iface
0.0.0.0 192.168.227.1 0.0.0.0 UG 0 0 0 eth0
192.168.227.0 0.0.0.0 255.255.255.0 U 0 0 0 eth0
```

### 不同主机间容器通信

不同容器之间的通信可以借助于`pipework`这个工具：
 
``` bash
$ git clone https://github.com/jpetazzo/pipework.git
$ sudo cp -rp pipework/pipework /usr/local/bin/
```
 
#### 安装相应依赖软件
 
``` bash
$ sudo apt-get install apring bridge-utils -y
```
 
#### 桥接网络
 
Ubuntu14.04
 
``` bash
# cat /etc/network/interfaces
auto lo
iface lo inet loopback
 
auto eth0
iface eth0 inet manual
 
auto br0
iface br0 inet static
address 10.0.128.219
netmask 255.255.255.192
gateway 10.0.128.254
bridge_ports eth0
bridge_stp off
bridge_fd 0
bridge_maxwait 0
dns-nameservers 10.0.127.110
dns-search intranet.123u.com
```
 
##### 启动br0，使桥接生效
 
``` bash
# ifup br0
# Bash=$(docker run -i -d -t 10.0.128.219:5000/ubuntu:14.04 /bin/bash)
# pipework br0 $Bash 10.0.128.223/26
```

### 参考文档

* [pipework readme](https://github.com/jpetazzo/pipework/blob/master/README.md)
* [pipework-docker网络增强工具](http://peerxu.github.io/blog/2014/04/07/docker-with-openvswitch.html)

## 构建docker私有库

为方便管理，我们需要对官方的镜像做一些定制，我们可以构建私有的`docker registry`

### 快速构建
 
The fastest way to get running:
 
* install docker：`apt-get install docker.io`
* run the registry: `docker run -p 5000:5000 registry`
 
That will use the official image from the Docker index.[因为国内被墙的原因，速度比较慢，推荐第二种方式]
 
### 传统构建方式
 
``` bash
$ sudo apt-get install build-essential python-dev libevent-dev python-pip liblzma-dev
$ git clone https://github.com/dotcloud/docker-registry.git
$ cd docker-registry/
$ cp config/config_sample.yml config/config.yml
$ mkdir /data/registry -p
$ pip install .
```
 
#### 启动
 
``` bash
$ sudo gunicorn --access-logfile - --debug -k gevent -b 0.0.0.0:5000 \
-w 1 docker_registry.wsgi:application
```

生产环境可以通过如supervisord创建8个workers，或者通过nginx和apache来管理，具体可以参考[docker-registry readme](https://github.com/dotcloud/docker-registry)。

``` bash
$ sudo gunicorn -k gevent --max-requests 100 --graceful-timeout 3600 \
-t 3600 -b localhost:5000 -w 8 docker_registry.wsgi:application
```
 
#### 提交指定容器到私有库
 
``` bash
$ docker tag 74fe38d11401 192.168.0.219:5000/ubuntu:12.04
$ docker push 192.168.0.219:5000/ubuntu
```

### 参考

* [docker-registry readme](https://github.com/dotcloud/docker-registry)
* [How to use your own Registry](http://blog.docker.io/2013/07/how-to-use-your-own-registry/)