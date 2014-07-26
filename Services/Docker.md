# 轻量级虚拟化Docker

## 一、Docker基本介绍

Docker发端于一个名为dotcloud的开源项目；随着编写者不断挖掘它的潜力，它迅速变成了一个炙手可热的项目。它由GO语言编写的，并且只支持Linux。它基于Linux容器（LxC）来创建一个虚拟环境。Docker不会通过建立独有的操作系统、进程和对硬件进行模拟来创建属于自己的虚拟机。请注意：虚拟环境VE(Virtual Environment)和虚拟机（VM）很不一样。虚拟机是由虚拟工具或者模拟器（HyperV 、VMWare等）创建的，是一个全镜像的主机源，其中包括操作系统、硬盘调整、网络和虚拟进程。过于臃肿的结构吃掉了大量的硬盘空间同时拖慢了运行和开机速度。
 
一台VE就像是轻量级的VM，它在已有的内核关于底层硬件的镜像上建立一个可以用来运行应用的‘容器’。它也可以用来创建操作系统，因为所谓的操作系统也不过是一个跑在内核上的应用而已。可以把Docker想象成LxC的一个强化版，只是具有以下LxC所不具有的特性：
 
- **强大的可移植性**：你可以使用Docker创造一个绑定了你所有你所需要的应用的对象。这个对象可以被转移并被安装在任何一个安装了 Docker 的 Linux 主机上。
- **版本控制**： Docker自带git功能，能够跟踪一个容器的成功版本并记录下来，并且可以对不同的版本进行检测，提交新版本，回滚到任意的一个版本等功能等等。
- **组件的可重用性**： Docker 允许创建或是套用一个已经存在的包。举个例子，如果你有许多台机器都需要安装 Apache 和 MySQL 数据库，你可以创建一个包含了这两个组件的‘基础镜像’。然后在创建新机器的时候使用这个镜像进行安装就行了。
- **可分享的类库**：已经有上千个可用的容器被上传并被分享到一个共有仓库中[registry.hub.docker.com](https://registry.hub.docker.com/)。考虑到AWS对于不同环境下的调试和发布，这一做法是十分聪明的。
 
> LxC是一个Linux提供的收容功能接口，通过LxC提供的API和简单的工具，使得Linux用户可以简单的创建和管理系统或者应用的空间。[LXC容器](http://www.ibm.com/developerworks/cn/linux/l-lxc-containers/)
 
Docker通常用于如下场景：
 
    web应用的自动化打包和发布；
    自动化测试和持续集成、发布；
    在服务型环境中部署和调整数据库或其他的后台应用；
    从头编译或者扩展现有的OpenShift或Cloud Foundry平台来搭建自己的PaaS环境。
 
Docker实践解决方案：
 
- 隔离性：Docker在文件系统和网络级别隔离了应用。从这个意义上来讲很像在运行”真正的“虚拟机。
- 重复性：用你喜欢的方式准备系统（登录并在所有软件里执行apt-get命令，或者使用Dockerfile），然后把修改提交到镜像中。你可以随意实例化若干个实例，或者把镜像传输到另一台机器，完全重现同样的设置。
- 安全性：Docker容器比普通的进程隔离更为安全。Docker团队已经确定了一些安全问题，正在着手解决。
- 资源约束：Docker现在能限制CPU的使用率和内存用量。目前还不能直接限制磁盘的使用情况。
- 易于安装：Docker有一个Docker Index，这个仓库存储了现成的Docker镜像，你用一条命令就可以完成实例化。比如说，要使用Clojure REPL镜像，只要运行docker run -t -i zefhemel/clojure-repl命令就能自动获取并运行该镜像。
- 易于移除：不需要应用了？销毁容器就行。
- 升级、降级：和EC2VM一样：先启动应用的新版本，然后把负载均衡器切换到新的端口。
- 快照、备份：Docker能提交镜像并给镜像打标签，和EC2上的快照不同，Docker是立即处理的。
 
参考文档：

- [Docker Getting Start: Related Knowledge ](http://tiewei.github.io/cloud/Docker-Getting-Start/)
- [谁是容器中的“战斗机”？Docker与Chef、LXC等容器对比](http://code.csdn.net/news/2819773)
- [Docker：利用Linux容器实现可移植的应用部署](http://www.infoq.com/cn/articles/docker-containers)

## 二、docker 安装配置

### 2.1、Docker install
 
Docker的安装非常简单，这里只介绍Ubuntu 14.04的安装，其他发行版本的安装可以参考官网手册。
 
``` bash
$ sudo apt-get update
$ sudo apt-get install docker.io
$ sudo ln -sf /usr/bin/docker.io /usr/local/bin/docker
```
 
获取当前docker版本
 
``` bash
$ sudo docker version
Client version: 1.1.1
Client API version: 1.13
Go version (client): go1.2.1
Git commit (client): bd609d2
Server version: 1.1.1
Server API version: 1.13
Go version (server): go1.2.1
Git commit (server): bd609d2
```
 
- [Docker install on ubuntu](http://docs.docker.io/installation/ubuntulinux/)
 
## 三、Docker images
 
- [Docker index](https://registry.hub.docker.com/) Docker镜像首页，包括官方镜像和其它公开镜像
 
### 3.1、Search index images
 
``` bash
$ sudo docker search ubuntu
```
 
### 3.2、Pull images
 
``` bash
$ sudo docker pull ubuntu # remote index 获取ubuntu官方镜像
$ sudo docker images # 查看当前镜像列表
REPOSITORY          TAG                 IMAGE ID            CREATED             VIRTUAL SIZE
ubuntu              13.10               5e019ab7bf6d        3 weeks ago         180 MB
ubuntu              saucy               5e019ab7bf6d        3 weeks ago         180 MB
ubuntu              12.04               74fe38d11401        3 weeks ago         209.6 MB
... ...
```
 
### 3.3、Running an interactive shell
 
``` bash
$ sudo docker run -i -t ubuntu:14.04 /bin/bash
```

- docker run - 运行一个容器
- -t - 分配一个（伪）tty (link is external)
- -i - 交互模式(so we can interact with it)
- ubuntu - 使用ubuntu基础镜像
- /bin/bash - 运行bash shell

 
__注:__ ubuntu会有多个版本，通过指定tag来启动特定的版本[image]:[tag]
 
``` bash
$ sudo docker ps # 查看当前运行的容器, ps -a列出当前系统所有的容器
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
6c9129e9df10        ubuntu:14.04        /bin/bash           6 minutes ago       Up 6 minutes                            cranky_babbage
```

### 3.4、相关快捷键

- 退出：`Ctrl-D` or `exit`
- detach：`Ctrl-p + Ctrl-q`
- attach: `docker attach CONTAINER ID`

## 四、docker常用命令

### 4.1、docker help

``` bash
$ sudo docker   # docker命令帮助
Usage: docker [OPTIONS] COMMAND [arg...]
 -H=[unix:///var/run/docker.sock]: tcp://host:port to bind/connect to or unix://path/to/socket to use

A self-sufficient runtime for linux containers.

Commands:
    attach    Attach to a running container                 # 当前shell下attach连接指定运行镜像
    build     Build an image from a Dockerfile              # 通过Dockerfile定制镜像
    commit    Create a new image from a container's changes # 提交当前容器为新的镜像
    cp        Copy files/folders from the containers filesystem to the host path
              # 从容器中拷贝指定文件或者目录到宿主机中
    diff      Inspect changes on a container's filesystem   # 查看docker容器变化
    events    Get real time events from the server          # 从docker服务获取容器实时事件
    export    Stream the contents of a container as a tar archive   
              # 导出容器的内容流作为一个tar归档文件[对应import]
    history   Show the history of an image                  # 展示一个镜像形成历史
    images    List images                                   # 列出系统当前镜像
    import    Create a new filesystem image from the contents of a tarball  
              # 从tar包中的内容创建一个新的文件系统映像[对应export]
    info      Display system-wide information               # 显示系统相关信息
    inspect   Return low-level information on a container   # 查看容器详细信息
    kill      Kill a running container                      # kill指定docker容器
    load      Load an image from a tar archive              # 从一个tar包中加载一个镜像[对应save]
    login     Register or Login to the docker registry server   
              # 注册或者登陆一个docker源服务器
    logs      Fetch the logs of a container                 # 输出当前容器日志信息
    port      Lookup the public-facing port which is NAT-ed to PRIVATE_PORT
              # 查看映射端口对应的容器内部源端口
    pause     Pause all processes within a container        # 暂停容器
    ps        List containers                               # 列出容器列表
    pull      Pull an image or a repository from the docker registry server
              # 从docker镜像源服务器拉取指定镜像或者库镜像
    push      Push an image or a repository to the docker registry server
              # 推送指定镜像或者库镜像至docker源服务器
    restart   Restart a running container                   # 重启运行的容器
    rm        Remove one or more containers                 # 移除一个或者多个容器
    rmi       Remove one or more images                 
              # 移除一个或多个镜像[无容器使用该镜像才可删除，否则需删除相关容器才可继续或-f强制删除]
    run       Run a command in a new container
              # 在一个新的容器中运行一个命令
    save      Save an image to a tar archive                # 保存一个镜像为一个tar包[对应load]
    search    Search for an image in the docker index       # 在docker index中搜索镜像
    start     Start a stopped containers                    # 启动容器
    stop      Stop a running containers                     # 停止容器
    tag       Tag an image into a repository                # 给源中镜像打标签
    top       Lookup the running processes of a container   # 查看容器中运行的进程信息
    unpause   Unpause a paused container                    # 取消暂停容器
    version   Show the docker version information           # 查看docker版本号
    wait      Block until a container stops, then print its exit code   
              # 截取容器停止时的退出状态值
```


docker选项帮助

``` bash
$ sudo docker --help
Usage of docker:
  --api-enable-cors=false                Enable CORS headers in the remote API                      # 远程API中开启CORS头
  -b, --bridge=""                        Attach containers to a pre-existing network bridge         # 桥接网络
                                           use 'none' to disable container networking
  --bip=""                               Use this CIDR notation address for the network bridge's IP, not compatible with -b
                                         # 和-b选项不兼容，具体没有测试过
  -d, --daemon=false                     Enable daemon mode                                         # daemon模式
  -D, --debug=false                      Enable debug mode                                          # debug模式
  --dns=[]                               Force docker to use specific DNS servers                   # 强制docker使用指定dns服务器
  --dns-search=[]                        Force Docker to use specific DNS search domains            # 强制docker使用指定dns搜索域
  -e, --exec-driver="native"             Force the docker runtime to use a specific exec driver     # 强制docker运行时使用指定执行驱动器
  -G, --group="docker"                   Group to assign the unix socket specified by -H when running in daemon mode
                                           use '' (the empty string) to disable setting of a group
  -g, --graph="/var/lib/docker"          Path to use as the root of the docker runtime              # 容器运行的根目录路径
  -H, --host=[]                          The socket(s) to bind to in daemon mode                    # daemon模式下docker指定绑定方式[tcp or 本地socket]
                                           specified using one or more tcp://host:port, unix:///path/to/socket, fd://* or fd://socketfd.
  --icc=true                             Enable inter-container communication                       # 跨容器通信
  --ip="0.0.0.0"                         Default IP address to use when binding container ports     # 指定监听地址，默认所有ip
  --ip-forward=true                      Enable net.ipv4.ip_forward                                 # 开启转发
  --iptables=true                        Enable Docker's addition of iptables rules                 # 添加对应iptables规则
  --mtu=0                                Set the containers network MTU                             # 设置网络mtu
                                           if no value is provided: default to the default route MTU or 1500 if no default route is available
  -p, --pidfile="/var/run/docker.pid"    Path to use for daemon PID file                            # 指定pid文件位置
  -r, --restart=true                     Restart previously running containers                      # 重新启动以前运行的容器                     
  -s, --storage-driver=""                Force the docker runtime to use a specific storage driver  # 强制docker运行时使用指定存储驱动
  --selinux-enabled=false                Enable selinux support                                     # 开启selinux支持
  --storage-opt=[]                       Set storage driver options                                 # 设置存储驱动选项
  --tls=false                            Use TLS; implied by tls-verify flags                       # 开启tls
  --tlscacert="/root/.docker/ca.pem"     Trust only remotes providing a certificate signed by the CA given here
  --tlscert="/root/.docker/cert.pem"     Path to TLS certificate file                               # tls证书文件位置
  --tlskey="/root/.docker/key.pem"       Path to TLS key file                                       # tls key文件位置
  --tlsverify=false                      Use TLS and verify the remote (daemon: verify client, client: verify daemon) # 使用tls并确认远程控制主机
  -v, --version=false                    Print version information and quit                         # 输出docker版本信息
```

#### 4.1.1、docker search

官方镜像源地址：[registry.hub.docker.com](https://registry.hub.docker.com/)

``` bash
$ sudo docker search

Usage: docker search TERM

Search the docker index for images      # 从docker镜像主页搜索镜像

  --automated=false    Only show automated builds
  --no-trunc=false     Don't truncate output
  -s, --stars=0        Only displays with at least xxx stars
```

示例：

``` bash
$ sudo docker search -s 100 ubuntu      
# 查找star数至少为100的镜像，找出只有官方镜像start数超过100，默认不加s选项找出所有相关ubuntu镜像
NAME      DESCRIPTION                  STARS     OFFICIAL   AUTOMATED
ubuntu    Official Ubuntu base image   425       [OK]       
```

####  4.1.2、docker info

``` bash
$ sudo docker info 
Containers: 7                       # 容器个数
Images: 102                         # 镜像个数
Storage Driver: aufs                # 存储驱动，默认aufs
 Root Dir: /var/lib/docker/aufs     # 根目录
 Dirs: 116
Execution Driver: native-0.2        # 执行驱动
Kernel Version: 3.13.0-24-generic
WARNING: No swap limit support
```

####  4.1.3、docker pull && docker push

``` bash
$ sudo docker pull                  # pull拉取镜像

Usage: docker pull NAME[:TAG]

Pull an image or a repository from the registry

$ sudo docker push                  # push推送指定镜像

Usage: docker push NAME[:TAG]

Push an image or a repository to the registry
```

示例：

``` bash
$ sudo docker pull ubuntu           # 下载官方ubuntu docker镜像，默认下载所有ubuntu官方库镜像
$ sudo docker pull ubuntu:14.04     # 下载指定版本ubuntu官方镜像
```

``` bash
$ sudo docker push 192.168.0.100:5000/ubuntu
# 推送镜像库到私有源[可注册docker官方账户，推送到官方自有账户]
$ sudo docker push 192.168.0.100:5000/ubuntu:14.04 
# 推送指定镜像到私有源
```

#### 4.1.4、docker images

列出当前系统镜像

``` bash
$ sudo docker images -h

Usage: docker images [OPTIONS] [NAME]

List images

  -a, --all=false      Show all images (by default filter out the intermediate image layers)
  # -a显示当前系统的所有镜像，包括过渡层镜像，默认docker images显示最终镜像，不包括过渡层镜像
  -f, --filter=[]      Provide filter values (i.e. 'dangling=true')
  --no-trunc=false     Don't truncate output
  -q, --quiet=false    Only show numeric IDs
```

示例：

``` bash
$ sudo docker images            # 显示当前系统镜像，不包括过渡层镜像
$ sudo docker images -a         # 显示当前系统所有镜像，包括过渡层镜像
$ sudo docker images ubuntu     # 显示当前系统docker ubuntu库中的所有镜像
REPOSITORY                 TAG                 IMAGE ID            CREATED             VIRTUAL SIZE
ubuntu                     12.04               ebe4be4dd427        4 weeks ago         210.6 MB
ubuntu                     14.04               e54ca5efa2e9        4 weeks ago         276.5 MB
ubuntu                     14.04-ssh           6334d3ac099a        7 weeks ago         383.2 MB
```

#### 4.1.5、docker rmi

删除一个或者多个镜像

``` bash
$ sudo docker rmi

Usage: docker rmi IMAGE [IMAGE...]

Remove one or more images

  -f, --force=false    Force removal of the image       # 强制移除镜像不管是否有容器使用该镜像
  --no-prune=false     Do not delete untagged parents   # 不要删除未标记的父镜像
``` 

#### 4.1.6、docker run

``` bash
$ sudo docker run 

Usage: docker run [OPTIONS] IMAGE [COMMAND] [ARG...]

Run a command in a new container

  -a, --attach=[]            Attach to stdin, stdout or stderr.
  -c, --cpu-shares=0         CPU shares (relative weight)                       # 设置cpu使用权重
  --cidfile=""               Write the container ID to the file                 # 把容器id写入到指定文件
  --cpuset=""                CPUs in which to allow execution (0-3, 0,1)        # cpu绑定
  -d, --detach=false         Detached mode: Run container in the background, print new container id # 后台运行容器
  --dns=[]                   Set custom dns servers                             # 设置dns
  --dns-search=[]            Set custom dns search domains                      # 设置dns域搜索
  -e, --env=[]               Set environment variables                          # 定义环境变量
  --entrypoint=""            Overwrite the default entrypoint of the image      # ？
  --env-file=[]              Read in a line delimited file of ENV variables     # 从指定文件读取变量值
  --expose=[]                Expose a port from the container without publishing it to your host    # 指定对外提供服务端口
  -h, --hostname=""          Container host name                                # 设置容器主机名
  -i, --interactive=false    Keep stdin open even if not attached               # 保持标准输出开启即使没有attached
  --link=[]                  Add link to another container (name:alias)         # 添加链接到另外一个容器[这个会专门章节讲解]
  --lxc-conf=[]              (lxc exec-driver only) Add custom lxc options --lxc-conf="lxc.cgroup.cpuset.cpus = 0,1"
  -m, --memory=""            Memory limit (format: <number><optional unit>, where unit = b, k, m or g) # 内存限制
  --name=""                  Assign a name to the container                     # 设置容器名
  --net="bridge"             Set the Network mode for the container             # 设置容器网络模式
                               'bridge': creates a new network stack for the container on the docker bridge
                               'none': no networking for this container
                               'container:<name|id>': reuses another container network stack
                               'host': use the host network stack inside the container.  Note: the host mode gives the container full access to local system services such as D-bus and is therefore considered insecure.
  -P, --publish-all=false    Publish all exposed ports to the host interfaces   # 自动映射容器对外提供服务的端口
  -p, --publish=[]           Publish a container's port to the host             # 指定端口映射
                               format: ip:hostPort:containerPort | ip::containerPort | hostPort:containerPort
                               (use 'docker port' to see the actual mapping)
  --privileged=false         Give extended privileges to this container         # 提供更多的权限给容器
  --rm=false                 Automatically remove the container when it exits (incompatible with -d) # 如果容器退出自动移除和-d选项冲突
  --sig-proxy=true           Proxify received signals to the process (even in non-tty mode). SIGCHLD is not proxied. # ？
  -t, --tty=false            Allocate a pseudo-tty                              # 分配伪终端
  -u, --user=""              Username or UID                                    # 指定运行容器的用户uid或者用户名
  -v, --volume=[]            Bind mount a volume (e.g., from the host: -v /host:/container, from docker: -v /container)     
                             # 挂载卷[这个会专门章节讲解]
  --volumes-from=[]          Mount volumes from the specified container(s)      # 从指定容器挂载卷
  -w, --workdir=""           Working directory inside the container             # 指定容器工作目录
```

示例：

``` bash
$ sudo docker images ubuntu
REPOSITORY          TAG                 IMAGE ID            CREATED             VIRTUAL SIZE
ubuntu              14.04               e54ca5efa2e9        4 weeks ago         276.5 MB
... ...
$ sudo docker run -t -i -c 100 -m 512MB -h test1 -d --name="docker_test1" ubuntu /bin/bash 
# 创建一个cpu优先级为1，内存限制512MB，主机名为test1，名为docker_test1后台运行bash的容器
a424ca613c9f2247cd3ede95adfbaf8d28400cbcb1d5f9b69a7b56f97b2b52e5
$ sudo docker ps 
CONTAINER ID        IMAGE           COMMAND         CREATED             STATUS              PORTS       NAMES
a424ca613c9f        ubuntu:14.04    /bin/bash       6 seconds ago       Up 5 seconds                    docker_test1
$ sudo docker attach docker_test1
root@test1:/# pwd
/
root@test1:/# exit
exit
```

__关于cpu优先级:__

> By default all groups have 1024 shares. A group with 100 shares will get a ~10% portion of the CPU time:

#### 4.1.7、docker start|stop|kill... ...

docker `start`|`stop`|`kill`|`restart`|`pause`|`unpause`|`rm`|`commit`|`inspect`

* docker start CONTAINER [CONTAINER...] # 运行一个或多个停止的容器
* docker stop CONTAINER [CONTAINER...]  # 停掉一个或多个运行的容器 `-t`选项可指定超时时间 
* docker kill [OPTIONS] CONTAINER [CONTAINER...] # 默认kill发送SIGKILL信号 `-s`可以指定发送kill信号类型
* docker restart [OPTIONS] CONTAINER [CONTAINER...] # 重启一个或多个运行的容器 `-t`选项可指定超时时间
* docker pause CONTAINER                        # 暂停一个容器，方便commit
* docker unpause CONTAINER                      # 继续暂停的容器
* docker rm [OPTIONS] CONTAINER [CONTAINER...]  # 移除一个或多个容器
    * -f, --force=false      Force removal of running container
    * -l, --link=false       Remove the specified link and not the underlying container 
    * -v, --volumes=false    Remove the volumes associated with the container
* docker commit [OPTIONS] CONTAINER [REPOSITORY[:TAG]]  # 提交指定容器为镜像
    * -a, --author=""     Author (e.g., "John Hannibal Smith <hannibal@a-team.com>")
    * -m, --message=""    Commit message
    * -p, --pause=true    Pause container during commit # 默认commit是暂停状态
* docker inspect CONTAINER|IMAGE [CONTAINER|IMAGE...]   # 查看容器或者镜像的详细信息


### 4.2、参考文档

* [Docker Run Reference](https://docs.docker.com/reference/run/)

## 五、docker端口映射

``` bash
# Find IP address of container with ID <container_id> 通过容器id获取ip
$ sudo docker inspect <container_id> | grep IPAddress | cut -d ’"’ -f 4
```
 
无论如何，这些ip是基于本地系统的并且容器的端口非本地主机是访问不到的。此外，除了端口只能本地访问外，对于容器的另外一个问题是这些ip在容器每次启动的时候都会改变。
 
Docker解决了容器的这两个问题，并且给容器内部服务的访问提供了一个简单而可靠的方法。Docker通过端口绑定主机系统的接口，允许非本地客户端访问容器内部运行的服务。为了简便的使得容器间通信，Docker提供了这种连接机制。
 
### 5.1、自动映射端口
 
`-P`使用时需要指定`--expose`选项，指定需要对外提供服务的端口
 
``` bash
$ sudo docker run -t -P --expose 22 --name server  ubuntu:14.04
```
 
使用`docker run -P`自动绑定所有对外提供服务的容器端口，映射的端口将会从没有使用的端口池中(49000..49900)自动选择，你可以通过`docker ps`、`docker inspect <container_id>`或者`docker port <container_id> <port>`确定具体的绑定信息。
 
### 5.2、绑定端口到指定接口
 
基本语法
 
``` bash
$ sudo docker run -p [([<host_interface>:[host_port]])|(<host_port>):]<container_port>[/udp] <image> <cmd>
```
 
默认不指定绑定ip则监听所有网络接口。
 
#### 5.2.1、绑定TCP端口
 
``` bash
# Bind TCP port 8080 of the container to TCP port 80 on 127.0.0.1 of the host machine.
$ sudo docker run -p 127.0.0.1:80:8080 <image> <cmd>
# Bind TCP port 8080 of the container to a dynamically allocated TCP port on 127.0.0.1 of the host machine.
$ sudo docker run -p 127.0.0.1::8080 <image> <cmd>
# Bind TCP port 8080 of the container to TCP port 80 on all available interfaces of the host machine.
$ sudo docker run -p 80:8080 <image> <cmd>
# Bind TCP port 8080 of the container to a dynamically allocated TCP port on all available interfaces
$ sudo docker run -p 8080 <image> <cmd>
```
 
#### 5.2.2、绑定UDP端口
 
``` bash
# Bind UDP port 5353 of the container to UDP port 53 on 127.0.0.1 of the host machine.
$ sudo docker run -p 127.0.0.1:53:5353/udp <image> <cmd>
```

## 六、配置网络

Docker uses Linux bridge capabilities to provide network connectivity to containers. The docker0 bridge interface is managed by Docker for this purpose. When the Docker daemon starts it :
 
Dokcer通过使用Linux桥接提供容器之间的通信，docker0桥接接口的目的就是方便Docker管理。当Docker daemon启动时需要做以下操作：
 
- creates the docker0 bridge if not present 如果docker0不存在则创建
- searches for an IP address range which doesn’t overlap with an existing route 搜索一个与当前路由不冲突的ip段
- picks an IP in the selected range 在确定的范围中选择ip
- assigns this IP to the docker0 bridge 绑定ip到docker0
 
### 6.1、列出当前主机网桥
 
``` bash
$ sudo brctl show  # brctl工具依赖bridge-utils软件包
bridge name bridge id STP enabled interfaces
docker0 8000.000000000000 no
```
 
### 6.2、查看当前docker0 ip
 
``` bash
$ sudo ifconfig docker0
docker0 Link encap:Ethernet HWaddr xx:xx:xx:xx:xx:xx
inet addr:172.17.42.1 Bcast:0.0.0.0 Mask:255.255.0.0
```
 
在容器运行时，每个容器都会分配一个特定的虚拟机口并桥接到docker0。每个容器都会配置同docker0 ip相同网段的专用ip地址，docker0 的IP地址被用于所有容器的默认网关。
 
### 6.3、运行一个容器
 
``` bash
$ sudo docker run -t -i -d ubuntu /bin/bash
52f811c5d3d69edddefc75aff5a4525fc8ba8bcfa1818132f9dc7d4f7c7e78b4
$ sudo brctl show
bridge name bridge id STP enabled interfaces
docker0 8000.fef213db5a66 no vethQCDY1N
```
 
以上, docker0 扮演着52f811c5d3d6 container这个容器的虚拟接口vethQCDY1N interface桥接的角色。
 
#### 6.3.1、使用特定范围的IP
 
Docker会尝试寻找没有被主机使用的ip段，尽管它适用于大多数情况下，但是它不是万能的，有时候我们还是需要对ip进一步的规划。Docker允许你管理docker0桥接或者通过`-b`选项自定义桥接网卡，需要安装`bridge-utils`软件包。
 
基本步骤如下：
 
- ensure Docker is stopped  确保docker的进程是停止的
- create your own bridge (bridge0 for example) 创建自定义网桥
- assign a specific IP to this bridge   给网桥分配特定的ip
- start Docker with the -b=bridge0 parameter    以-b的方式指定网桥
 
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

### 6.4、不同主机间容器通信

不同容器之间的通信可以借助于`pipework`这个工具：
 
``` bash
$ git clone https://github.com/jpetazzo/pipework.git
$ sudo cp -rp pipework/pipework /usr/local/bin/
```
 
#### 6.4.1、安装相应依赖软件
 
``` bash
$ sudo apt-get install apring bridge-utils -y
```
 
#### 6.4.2、桥接网络
 
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

### 6.5、参考文档

- [pipework readme](https://github.com/jpetazzo/pipework/blob/master/README.md)
- [pipework-docker网络增强工具](http://peerxu.github.io/blog/2014/04/07/docker-with-openvswitch.html)

## 七、构建docker私有库

为方便管理，我们需要对官方的镜像做一些定制，我们可以构建私有的`docker registry`

### 7.1、快速构建
 
The fastest way to get running:
 
- install docker：`apt-get install docker.io`
- run the registry: `docker run -p 5000:5000 registry`
 
That will use the official image from the Docker index.[因为国内被墙的原因，速度比较慢，推荐第二种方式]
 
### 7.2、传统构建方式
 
``` bash
$ sudo apt-get install build-essential python-dev libevent-dev python-pip liblzma-dev
$ git clone https://github.com/dotcloud/docker-registry.git
$ cd docker-registry/
$ cp config/config_sample.yml config/config.yml
$ mkdir /data/registry -p
$ pip install .
```
 
#### 7.2.1、启动
 
``` bash
$ sudo gunicorn --access-logfile - --debug -k gevent -b 0.0.0.0:5000 \
-w 1 docker_registry.wsgi:application
```

生产环境可以通过如`supervisord`创建8个workers，或者通过nginx和apache来管理，具体可以参考[docker-registry readme](https://github.com/dotcloud/docker-registry)。

``` bash
$ sudo gunicorn -k gevent --max-requests 100 --graceful-timeout 3600 \
-t 3600 -b localhost:5000 -w 8 docker_registry.wsgi:application
```
 
#### 7.2.2、提交指定容器到私有库
 
``` bash
$ docker tag 74fe38d11401 192.168.0.219:5000/ubuntu:12.04
$ docker push 192.168.0.219:5000/ubuntu
```

### 7.3、参考

- [docker-registry readme](https://github.com/dotcloud/docker-registry)
- [How to use your own Registry](http://blog.docker.io/2013/07/how-to-use-your-own-registry/)

## 八、容器数据管理

docker管理数据的方式有两种：

* 数据卷
* 数据卷容器

### 数据卷

数据库是一个或多个容器专门指定绕过`Union File System`的目录，为持续性或共享数据提供一些有用的功能：

* 数据卷可以在容器间共享和重用
* 数据卷数据改变是直接修改的
* 数据卷数据改变不会被包括在容器中
* 数据卷是持续性的，直到没有容器使用它们

#### 添加一个数据卷

你可以使用`-v`选项添加一个数据卷，或者可以使用多次`-v`选项为一个docker容器运行挂载多个数据卷。

``` bash
$ sudo docker run --name data -v /data -t -i centos:6.4 /bin/bash
# 宿主机/data数据卷绑定到到新建容器，新建容器中会创建/data数据卷
bash-4.1# ls -ld /data/
drwxr-xr-x 2 root root 4096 Jul 23 06:59 /data/
bash-4.1# df -Th
Filesystem    Type    Size  Used Avail Use% Mounted on
... ...
              ext4     91G  4.6G   82G   6% /data
```

创建的数据卷可以通过`docker inspect`获取

``` bash
$ sudo docker inspect data
... ...
    "Volumes": {
        "/data": "/var/lib/docker/vfs/dir/151de401d268226f96d824fdf444e77a4500aed74c495de5980c807a2ffb7ea9"
    }, # 可以看到创建的数据卷宿主机路径
    "VolumesRW": {
        "/data1": true
    }
... ...
```

#### 挂载宿主机目录为一个数据卷

`-v`选项除了可以创建卷，也可以挂载当前主机的一个目录到容器中。

``` bash
$ sudo docker run --name web -v /source/:/web -t -i centos:6.4 /bin/bash
bash-4.1# ls -ld /web/
drwxr-xr-x 2 root root 4096 Jul 23 06:59 /web/
bash-4.1# df -Th
... ...
              ext4     91G  4.6G   82G   6% /web
bash-4.1# exit
```

默认挂载卷是可读写的，可以在挂载时指定只读

``` bash
$ sudo docker run --rm --name test -v /source/:/test:ro -t -i centos:6.4 /bin/bash
```


