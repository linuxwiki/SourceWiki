
## 一、简介

Network Time Protocol（NTP） 是用来使计算机时间同步化的一种协议，它可以使计算机对其服务器或时钟源（如石英钟，GPS 等等)做同步化，它可以提供高精准度的时间校正（LAN 上与标准间差小于1毫秒，WAN上几十毫秒），且可使用加密确认的方式来防止恶毒的协议攻击。默认使用 `UDP 123端口`

NTP 提供准确时间，首先需要一个准确的 UTC 时间来源，NTP 获得 UTC 的时间来源可以从原子钟、天文台、卫星，也可从 Internet 上获取。时间服务器按照 NTP 服务器的等级传播，根据离外部 UTC 源的远近将所有服务器归入不用的层 (Stratum) 中。Stratum-1 在顶层由外部 UTC 接入，stratum-1 的时间服务器为整个系统的基础，Stratum 的总数限制在 15 以内。下图为 NTP 层次图：

<center><img src="/images/Network_Time_Protocol_servers_and_clients.png" alt="ntp" title="ntp" width="400" /></center>

## 二、NTP Server 安装配置

关于 NTP 服务器的安装，根据不同版本安装方法也不同。REDHAT 系统则可以使用 yum 安装，Ubuntu 系列可以使用 `apt-get` 安装，这里不做具体的介绍，主要详细介绍配置文件的信息。

对于 Centos 过滤注释和空行后，ntp 配置文件内容如下

```
# grep -vE '^#|^$' /etc/ntp.conf 
driftfile /var/lib/ntp/drift
restrict default kod nomodify notrap nopeer noquery 
restrict -6 default kod nomodify notrap nopeer noquery
restrict 127.0.0.1 
restrict -6 ::1
server 0.centos.pool.ntp.org
server 1.centos.pool.ntp.org
server 2.centos.pool.ntp.org
includefile /etc/ntp/crypto/pw
keys /etc/ntp/keys
```

### 2.1 配置选项说明

* `driftfile` 选项， 则指定了用来保存系统时钟频率偏差的文件。 ntpd 程序使用它来自动地补偿时钟的自然漂移， 从而使时钟即使在切断了外来时源的情况下， 仍能保持相当的准确度。另外，driftfile 选项也保存上一次响应所使用的 NTP 服务器的信息。 这个文件包含了 NTP 的内部信息， 它不应被任何其他进程修改。`无需更改`
* `restrict default kod nomodify notrap nopeer noquery`  默认拒绝所有NTP客户端的操作 [ restrict <IP 地址> <子网掩码>|<网段> [ignore|nomodiy|notrap|notrust|nknod ] 指定可以通信的IP地址和网段。如果没有指定选项，表示客户端访问NTP服务器没有任何限制
	* `ignore`:     关闭所有 NTP 服务
	* `nomodiy`:    表示客户端不能更改 NTP 服务器的时间参数，但可以通过 NTP 服务器进行时间同步
	* `notrust`:    拒绝没有通过认证的客户端
	* `knod`:       kod 技术科阻止 "Kiss of Death" 包（一种 DOS 攻击）对服务器的破坏，使用 knod 开启功能
	* `nopeer`:     不与其它同一层的 NTP 服务器进行同步
* `server [IP|FQDN|prefer]`指该服务器上层 NTP Server，使用 prefer 的优先级最高，没有使用 prefer 则按照配置文件顺序由高到低，默认情况下至少 15min 和上层 NTP 服务器进行时间校对
* `fudge`:          可以指定本地 NTP Server 层，如 `fudge 127.0.0.1 stratum 9`
* `broadcast 网段 子网掩码`:    指定 NTP 进行时间广播的网段，如`broadcast 192.168.1.255`
* `logfile`:        可以指定 NTP Server 日志文件

几个与NTP相关的配置文件:` /usr/share/zoneinfo/`、`/etc/sysconfig/clock`、`/etc/localtime`

* `/usr/share/zoneinfo/`:  存放时区文件目录
* `/etc/sysconfig/clock`:  指定当前系统时区信息
* `/etc/localtime`:        相应的时区文件

如果需要修改当前时区，则可以从 /usr/share/zoneinfo/ 目录拷贝相应时区文件覆盖 /etc/localtime 并修改 /etc/sysconfig/clock 即可

```
cp /usr/share/zoneinfo/Asia/Shanghai /etc/localtime
sed -i 's:ZONE=.*:ZONE="Asia/Shanghai":g' /etc/sysconfig/clock
```

## 三、相关命令

`ntpstat` 查看同步状态

```
# ntpstat 
synchronised to NTP server (192.168.0.18) at stratum 4 
   time correct to within 88 ms  	# 表面时间校正 88ms
   polling server every 1024 s		# 每隔 1024s 更新一次
```

`ntpq` 列出上层状态

```
# ntpq -np
     remote           refid      st t when poll reach   delay   offset  jitter
==============================================================================
*192.168.0.18       202.112.31.197   3 u  101 1024  377   14.268    0.998   0.143
```

输出说明：

* `remote`:  NTP Server
* `refid` :  参考的上层 ntp 地址
* `st`    :  层次
* `when`  :  上次更新时间距离现在时常
* `poll`  :  下次更新时间
* `reach` :  更新次数
* `delay` :  延迟
* `offset`:  时间补偿结果
* `jitter`:  与 BIOS 硬件时间差异

`ntpdate` 同步当前时间: `ntpdate NTP服务器地址`

## 四、参考文档

* [ NTP 百度百科](http://baike.baidu.com/view/60648.htm)
* [ NTP 维基百科](http://en.wikipedia.org/wiki/Network_Time_Protocol)
* 鸟哥 Linux 私房菜
