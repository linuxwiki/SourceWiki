# pssh && pdsh

## 一、pssh

`pssh` 是一个 python 编写可以在多台服务器上执行命令的工具，同时支持拷贝文件，是同类工具中很出色的，类似 `pdsh` 。为方便操作，使用前请在各个服务器上配置好密钥认证访问。项目地址: [parallel-ssh](https://code.google.com/p/parallel-ssh/)

### 1.1、安装

```
wget http://parallel-ssh.googlecode.com/files/pssh-2.3.1.tar.gz
tar zxvf pssh-2.3.1.tar.gz
cd pssh-2.3.1/
python setup.py install
```

### 1.2、pssh相关参数

* `pssh` 在多个主机上并行地运行命令
   * -h 执行命令的远程主机列表,文件内容格式 [user@]host[:port]
   		* 如 test@172.16.10.10:229
   * -H 执行命令主机，主机格式 user@ip:port 
   * -l 远程机器的用户名
   * -p 一次最大允许多少连接
   * -P 执行时输出执行信息
   * -o 输出内容重定向到一个文件
   * -e 执行错误重定向到一个文件
   * -t 设置命令执行超时时间
   * -A 提示输入密码并且把密码传递给ssh(如果私钥也有密码也用这个参数)
   * -O 设置ssh一些选项
   * -x 设置ssh额外的一些参数，可以多个，不同参数间空格分开
   * -X 同-x,但是只能设置一个参数
   * -i 显示标准输出和标准错误在每台host执行完毕后

### 1.3、附加工具

*	`pscp` 传输文件到多个 hosts，类似 `scp`
	* pscp -h hosts.txt -l irb2 foo.txt /home/irb2/foo.txt
*	`pslurp` 从多台远程机器拷贝文件到本地
*	`pnuke` 并行在远程主机杀进程
	* pnuke -h hosts.txt -l irb2 java
*	`prsync` 使用rsync协议从本地计算机同步到远程主机
	* prsync -r -h hosts.txt -l irb2 foo /home/irb2/foo

### 1.4、使用实例

写入主机到文件中，语法为 `用户名@主机ip`

```
$ cat host.txt 
root@192.168.230.128
wul@10.0.0.8
```

推荐使用 `-i` 选项输出信息而不是 `-P` 选项，`-h` 指定定义主机组

```
$ pssh -i -h host.txt 'date'
[1] 16:32:38 [SUCCESS] root@192.168.230.128
Mon Aug 12 16:32:38 CST 2013
[2] 16:32:38 [SUCCESS] wul@10.0.0.8
Mon Aug 12 16:32:38 CST 2013
```

`-x` 选项：指定 ssh 的一些额外选项

```
$ pssh -x '-t -t -o StrictHostKeyChecking=no' -i -h host.txt date
[1] 17:20:01 [SUCCESS] root@192.168.230.128
Mon Aug 12 17:20:01 CST 2013
Stderr: Connection to 192.168.230.128 closed.
[2] 17:20:01 [SUCCESS] wul@10.0.0.8
Mon Aug 12 17:20:01 CST 2013
Stderr: Connection to 10.0.0.8 closed.
```

`-H` 选项：指定单个主机

```
$ pssh -x '-t -t -o StrictHostKeyChecking=no' -i -H 192.168.230.128 -H wul@10.0.0.8 date
[1] 17:22:58 [SUCCESS] 192.168.230.128
Mon Aug 12 17:22:58 CST 2013
Stderr: Connection to 192.168.230.128 closed.
[2] 17:22:58 [SUCCESS] wul@10.0.0.8
Mon Aug 12 17:22:58 CST 2013
Stderr: Connection to 10.0.0.8 closed.
```

### 1.5、参考文档

* [pssh](http://linux.die.net/man/1/pssh) 
* [pssh-howto](http://www.theether.org/pssh/docs/0.2.3/pssh-HOWTO.html)

## 二、pdsh 

`pdsh`(Parallel Distributed SHell) 可并行的执行对目标主机的操作，对于批量执行命令和分发任务有很大的帮助，在使用前需要配置 ssh 无密码登录，[点击下载](http://sourceforge.net/projects/pdsh/)


### 2.1、pdsh 基本用法

```
 pdsh -h
Usage: pdsh [-options] command ...
-S                return largest of remote command return values
-h                output usage menu and quit                获取帮助
-V                output version information and quit       查看版本
-q                list the option settings and quit         列出 `pdsh` 执行的一些信息
-b                disable ^C status feature (batch mode)
-d                enable extra debug information from ^C status
-l user           execute remote commands as user           指定远程使用的用户
-t seconds        set connect timeout (default is 10 sec)   指定超时时间
-u seconds        set command timeout (no default)          类似 `-t`
-f n              use fanout of n nodes                     设置同时连接的目标主机的个数
-w host,host,...  set target node list on command line      指定主机，host 可以是主机名也可以是 ip
-x host,host,...  set node exclusion list on command line   排除某些或者某个主机
-R name           set rcmd module to name                   指定 rcmd 的模块名，默认使用 ssh
-N                disable hostname: labels on output lines  输出不显示主机名或者 ip
-L                list info on all loaded modules and exit  列出 `pdsh` 加载的模块信息
-a                target all nodes                          指定所有的节点
-g groupname      target hosts in dsh group "groupname"     指定 `dsh` 组名，编译安裝需要添加 -g 支持选项 `--with-dshgroups`
-X groupname      exclude hosts in dsh group "groupname"    排除组，一般和 -a 连用
available rcmd modules: exec,xcpu,ssh (default: ssh)        可用的执行命令模块，默认为 ssh
```

### 2.2、使用实例

#### 2.2.1、单个主机测试

```
$ pdsh -w 192.168.0.231 -l root uptime
192.168.0.231:  16:16:11 up 32 days, 22:14, ? users,  load average: 0.10, 0.14, 0.16
```

#### 2.2.2、多个主机测试

```
$ pdsh -w 192.168.0.[231-233] -l root uptime
192.168.0.233:  16:17:05 up 32 days, 22:17, ? users,  load average: 0.13, 0.12, 0.10
192.168.0.232:  16:17:05 up 32 days, 22:17, ? users,  load average: 0.45, 0.34, 0.27
192.168.0.231:  16:17:06 up 32 days, 22:15, ? users,  load average: 0.09, 0.13, 0.15
```

#### 2.2.3、逗号分隔主机

```
$ pdsh -w 192.168.0.231,192.168.0.234 -l root uptime
192.168.0.234:  16:19:44 up 32 days, 22:19, ? users,  load average: 0.17, 0.21, 0.20
192.168.0.231:  16:19:44 up 32 days, 22:17, ? users,  load average: 0.29, 0.18, 0.16
```

#### 2.2.4、`-x` 排除某个主机

```
$ pdsh -w 192.168.0.[231-233] -x 192.168.0.232 -l root uptime
192.168.0.233:  16:18:24 up 32 days, 22:19, ? users,  load average: 0.11, 0.12, 0.09
192.168.0.231:  16:18:25 up 32 days, 22:16, ? users,  load average: 0.11, 0.13, 0.15
```

#### 2.2.5、主机组
  
对于 -g 组，把对应的主机写入到 `/etc/dsh/group/` 或 `~/.dsh/group/` 目录下的文件中即可，文件名就是对应组名

```
$ cat ~/.dsh/group/dsh-test 
192.168.0.231
192.168.0.232
192.168.0.233
192.168.0.234
```

```
$ pdsh -g dsh-test -l root uptime
192.168.0.232:  16:21:38 up 32 days, 22:22, ? users,  load average: 0.01, 0.15, 0.21
192.168.0.231:  16:21:38 up 32 days, 22:19, ? users,  load average: 0.17, 0.16, 0.16
192.168.0.234:  16:21:39 up 32 days, 22:21, ? users,  load average: 0.15, 0.19, 0.19
192.168.0.233:  16:21:40 up 32 days, 22:22, ? users,  load average: 0.15, 0.15, 0.10
```

#### 2.2.6、`dshbak` 格式化输出

`pdsh` 的缺省输出格式为主机名加该主机的输出，在主机或输出多时会比较混乱，可以采用 `dshbak` 做一些格式化，让输出更清晰。

```
$ pdsh -g dsh-test -l root 'date'       # 查看哪些主机时间不一样，主机一多，可读性不强
192.168.0.232: Wed Jun 19 16:24:40 CST 2013
192.168.0.231: Wed Jun 19 16:24:40 CST 2013
192.168.0.234: Wed Jun 19 16:24:40 CST 2013
192.168.0.233: Wed Jun 19 16:24:40 CST 2013
```

使用 `dshbak` 之后可读性变得好了很多

```
$ pdsh -g dsh-test -l root 'date' | dshbak -c  
----------------
192.168.0.[231-232,234]
----------------
Wed Jun 19 16:24:18 CST 2013
----------------
192.168.0.233
----------------
Wed Jun 19 16:24:19 CST 2013
```

### 2.3、参考文档

* [Using PDSH](https://code.google.com/p/pdsh/wiki/UsingPDSH)