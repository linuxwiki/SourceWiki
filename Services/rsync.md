# rsync

## rsync基本介绍

`rsync`是类unix系统下的数据镜像备份工具，从软件的命名上就可以看出来了——remote sync。它的特性如下：


* 1、可以镜像保存整个目录树和文件系统
* 2、可以很容易做到保持原来文件的权限、时间、软硬链接等等
* 3、无须特殊权限即可安装
* 4、优化的流程，文件传输效率高
* 5、可以使用rsh、ssh等方式来传输文件，当然也可以通过直接的socket连接
* 6、支持匿名传输

在使用rsync 进行远程同步时，可以使用两种方式：__远程Shell方式__（用户验证由 ssh 负责）和 __C/S 方式__（即客户连接远程rsync服务器，用户验证由rsync服务器负责）。

无论本地同步目录还是远程同步数据，首次运行时将会把全部文件拷贝一次，以后再运行时将只拷贝有变化的文件（对于新文件）或文件的变化部分（对于原有文件）。

## rsync选项

``` bash
Usage: rsync [OPTION]... SRC [SRC]... DEST
  or   rsync [OPTION]... SRC [SRC]... [USER@]HOST:DEST
  or   rsync [OPTION]... SRC [SRC]... [USER@]HOST::DEST
  or   rsync [OPTION]... SRC [SRC]... rsync://[USER@]HOST[:PORT]/DEST
  or   rsync [OPTION]... [USER@]HOST:SRC [DEST]
  or   rsync [OPTION]... [USER@]HOST::SRC [DEST]
  or   rsync [OPTION]... rsync://[USER@]HOST[:PORT]/SRC [DEST]
```

* `-v` : Verbose (try -vv for more detailed information)            # 详细模式显示
* `-e` "ssh options" : specify the ssh as remote shell              # 指定ssh作为远程shell
* `-a` : archive mode   # 归档模式，表示以递归方式传输文件，并保持所有文件属性，等于-rlptgoD
    * `-r`(--recursive) : 目录递归
    * `-l`(--links) ：保留软链接
    * `-p`(--perms) ：保留文件权限
    * `-t`(--times) ：保留文件时间信息
    * `-g`(--group) ：保留属组信息
    * `-o`(--owner) ：保留文件属主信息
    * `-D`(--devices) ：保留设备文件信息
* `-z` : 压缩文件
* `-h` : 以可读方式输出
* `-H` : 复制硬链接
* `-X` : 保留扩展属性
* `-A` : 保留ACL属性
* `-n` : 只测试输出而不正真执行命令，推荐使用，特别防止`--delete`误删除！
* `--stats` : 输出文件传输的状态
* `--progress` : 输出文件传输的进度
* `--exclude-from` : 排除文件或目录
* `--numeric-ids` : 不映射uid/gid到user/group的名字
* `--delete` : 删除DST中SRC没有的文件，也就是所谓的镜像[mirror]备份

## 远程 Shell 方式


## rsync C/S 方式

--EOF--