# rsync

## 一、rsync基本介绍

`rsync`是类unix系统下的数据镜像备份工具，从软件的命名上就可以看出来了——remote sync。它的特性如下：


* 1、可以镜像保存整个目录树和文件系统
* 2、可以很容易做到保持原来文件的权限、时间、软硬链接等等
* 3、无须特殊权限即可安装
* 4、优化的流程，文件传输效率高
* 5、可以使用rsh、ssh等方式来传输文件，当然也可以通过直接的socket连接
* 6、支持匿名传输

在使用rsync 进行远程同步时，可以使用两种方式：__远程Shell方式__（用户验证由 ssh 负责）和 __C/S 方式__（即客户连接远程rsync服务器，用户验证由rsync服务器负责）。

无论本地同步目录还是远程同步数据，首次运行时将会把全部文件拷贝一次，以后再运行时将只拷贝有变化的文件（对于新文件）或文件的变化部分（对于原有文件）。

## 二、rsync选项

``` bash
Usage: rsync [OPTION]... SRC [SRC]... DEST
  or   rsync [OPTION]... SRC [SRC]... [USER@]HOST:DEST
  or   rsync [OPTION]... SRC [SRC]... [USER@]HOST::DEST
  or   rsync [OPTION]... SRC [SRC]... rsync://[USER@]HOST[:PORT]/DEST
  or   rsync [OPTION]... [USER@]HOST:SRC [DEST]
  or   rsync [OPTION]... [USER@]HOST::SRC [DEST]
  or   rsync [OPTION]... rsync://[USER@]HOST[:PORT]/SRC [DEST]
The ':' usages connect via remote shell, while '::' & 'rsync://' usages connect
to an rsync daemon, and require SRC or DEST to start with a module name.
```
  
__注:__ 在指定复制源时，路径是否有最后的 “/” 有不同的含义，例如：

* /data ：表示将整个 /data 目录复制到目标目录
* /data/ ：表示将 /data/ 目录中的所有内容复制到目标目录

### 2.1、常用选项

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
* `––exclude=PATTERN` : 指定排除一个不需要传输的文件匹配模式
* `––exclude-from=FILE` : 从 FILE 中读取排除规则
* `––include=PATTERN` : 指定需要传输的文件匹配模式
* `––include-from=FILE` : 从 FILE 中读取包含规则
* `--numeric-ids` : 不映射 uid/gid 到 user/group 的名字
* `-S, --sparse` : 对稀疏文件进行特殊处理以节省DST的空间
* `--delete` : 删除DST中SRC没有的文件，也就是所谓的镜像[mirror]备份

## 三、远程 Shell 方式

``` bash
rsync [OPTION]... SRC [SRC]... [USER@]HOST:DEST # 执行“推”操作
or   rsync [OPTION]... [USER@]HOST:SRC [DEST]   # 执行“拉”操作
```

## 四、rsync C/S 方式

``` bash
rsync [OPTION]... SRC [SRC]... [USER@]HOST::DEST                    # 执行“推”操作
or   rsync [OPTION]... SRC [SRC]... rsync://[USER@]HOST[:PORT]/DEST # 执行“推”操作
or   rsync [OPTION]... [USER@]HOST::SRC [DEST]                      # 执行“拉”操作
or   rsync [OPTION]... rsync://[USER@]HOST[:PORT]/SRC [DEST]        # 执行“拉”操作
```

C/S 方式需要配置服务端，下面是一个配置文件示例：

``` bash
# /etc/rsyncd.conf

uid = root
gid = root
use chroot = yes

[bak-data]
    path = /data/
    comment = data backup
    numeric ids = yes
    read only = yes
    list = no
    auth users = data
    filter = merge /etc/.data-filter  # 过滤规则
    secrets file = /etc/rsync-secret
    hosts allow = 192.168.80.0/24 172.16.0.10

[bak-home]
    path = /home/
    comment = home backup
    numeric ids = yes
    read only = yes
    list = no
    auth users = home,test
    exclude = .svn .git
    secrets file = /etc/rsync-secret
    hosts allow = 192.168.80.0/24 172.16.0.10
```

密码文件和 filter 文件内容如下：

``` bash
# cat /etc/rsync-secret
data:123321
home:123456
test:654321
# chmod 600 /etc/rsync-secret
# cat /etc/.data-filter     # 关于 filter 的规则文件需要多测试才能彻底明白
+ mysql56/***
- *
# 以上规则表示匹配所有 mysql56 目录下的内容，其它都不同步
```

关于filter的匹配规则可以参考[man手册](http://www.samba.org/ftp/rsync/rsyncd.conf.html)：

      filter
      The daemon has its own filter chain that determines what files it will let the client access. This chain is not sent to the client and is independent of any filters the client may have specified. Files excluded by the daemon filter chain (daemon-excluded files) are treated as non-existent if the client tries to pull them, are skipped with an error message if the client tries to push them (triggering exit code 23), and are never deleted from the module. You can use daemon filters to prevent clients from downloading or tampering with private administrative files, such as files you may add to support uid/gid name translations.

      The daemon filter chain is built from the "filter", "include from", "include", "exclude from", and "exclude" parameters, in that order of priority. Anchored patterns are anchored at the root of the module. To prevent access to an entire subtree, for example, "/secret", you must exclude everything in the subtree; the easiest way to do this is with a triple-star pattern like "/secret/***".

      The "filter" parameter takes a space-separated list of daemon filter rules, though it is smart enough to know not to split a token at an internal space in a rule (e.g. "- /foo - /bar" is parsed as two rules). You may specify one or more merge-file rules using the normal syntax. Only one "filter" parameter can apply to a given module in the config file, so put all the rules you want in a single parameter. Note that per-directory merge-file rules do not provide as much protection as global rules, but they can be used to make --delete work better during a client download operation if the per-dir merge files are included in the transfer and the client requests that they be used. 

## 五、一些命令

### 5.1、常用命令

``` bash
RSYNC_PASSWORD=123321 rsync -havAEHXi -n --numeric-ids --delete --stats --progress [SRC] [DEST]
```
  
__注：__ 如果有稀疏文件，则添加 `-S` 选项可以提升传输性能。

### 5.2、ssh端口非默认22同步

使用ssh方式传输时如果连接服务器ssh端口非标准，则需要通过`-e`选项指定：

``` bash
RSYNC_PASSWORD=123321 rsync -havAEHXi -n --numeric-ids --delete --stats --progress -e "ssh -p 22222" [USER@]HOST:SRC [DEST]
```

### 5.3、查看服务器同步资源

``` bash
RSYNC_PASSWORD=123321 rsync --list-only data@192.168.80.150::bak-data
或
RSYNC_PASSWORD=123321 rsync --list-only rsync://data@192.168.80.150/bak-data
```

## 六、参考文档

* [rsync man 手册](http://www.samba.org/ftp/rsync/rsyncd.conf.html)
* [howtocn rsync文档](http://www.howtocn.org/rsync:use_rsync)
* [使用 rsync 进行文件备份](http://blog.clanzx.net/2013/08/23/rsync-backup.html)

--EOF--