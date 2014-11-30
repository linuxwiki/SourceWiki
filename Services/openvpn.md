# openvpn基本搭建实例

OpenVPN 是一个用于创建虚拟专用网络加密通道的软件包，最早由 James Yonan 编写。OpenVPN 允许创建的 VPN 使用公开密钥、电子证书、或者用户名/密码来进行身份验证。

<center><img src="/images/openvpn.png" alt="openvpn" title="openvpn" width="500" /></center>

## 一、准备软件

本例以 CentOS 6 为例

* [openvpn-2.3.1](http://swupdate.openvpn.org/community/releases/openvpn-2.3.1.tar.gz)
* [lzo-2.06](http://www.oberhumer.com/opensource/lzo/download/lzo-2.06.tar.gz)
* [最新版本的 openvpn-2.3.1 官方不再集成`easy-rsa`] Note that easy-rsa is no longer bundled with OpenVPN source code archives. To get it, visit the easy-rsa page on GitHub, or download it from our Linux software repositories.

```
git clone https://github.com/OpenVPN/easy-rsa
```

## 二、软件安装

`openssl`、`openssl-devel`、`pam`、`pam-devel`安装

```
yum install openssl openssl-devel pam pam-devel -y
```

`lzo-2.06` 安装,下载相应软件包编译安装即可

```
./configure && make && make install
```

`openvpn-2.3.1` 安装，同上官网下载软件包编译安装

```
./configure && make && make install
```

## 三、相关配置

Openvpn 的认证方式有很多种，这里介绍其中一种，key 认证登录方式

### 添加环境变量

在 `~/.barc_profile` 文件中加入如下内容,命名根据实际需求修改：

```
export D=/etc/openvpn
export KEY_CONFIG=$D/openssl.cnf
export KEY_DIR=$D/keys
export KEY_SIZE=1024
export KEY_COUNTRY=CN
export KEY_PROVINCE=BJ
export KEY_CITY=BJ
export KEY_ORG=kumu
export KEY_OU=kumu
export KEY_NAME=kumu
export KEY_EMAIL=root@kumu
```

使新增环境变量生效并新建配置文件目录 `/etc/openvpn`

```
# source ~/.barc_profile
# mkdir /etc/openvpn
```

__注__:也可修改 easy-rsa 中的 vars [/usr/local/src/openvpn/easy-rsa/easy-rsa/2.0/vars]，`source` 生效

### 2.1 生成密钥

进入之前下载的 `easy-rsa` 目录

__初始化 PKI、生成证书__:

```
# pwd
/usr/local/src/openvpn/easy-rsa/easy-rsa/2.0
# ls
build-ca     build-key         build-key-server  clean-all      openssl-0.9.6.cnf  pkitool      vars
build-dh     build-key-pass    build-req         inherit-inter  openssl-0.9.8.cnf  revoke-full  whichopensslcnf
build-inter  build-key-pkcs12  build-req-pass    list-crl       openssl-1.0.0.cnf  sign-req
# cp openssl-1.0.0.cnf /etc/openvpn/openssl.cnf
# ./clean-all # 初始化，清除原有不需要的文件
# ./build-ca  # 一直回车即可
Generating a 1024 bit RSA private key
.......++++++
............++++++
writing new private key to 'ca.key'
-----
You are about to be asked to enter information that will be incorporated
into your certificate request.
What you are about to enter is what is called a Distinguished Name or a DN.
There are quite a few fields but you can leave some blank
For some fields there will be a default value,
If you enter '.', the field will be left blank.
-----
Country Name (2 letter code) [CN]:
State or Province Name (full name) [BJ]:
Locality Name (eg, city) [BJ]:
Organization Name (eg, company) [kumu]:
Organizational Unit Name (eg, section) [kumu]:
Common Name (eg, your name or your server's hostname) [kumu CA]:
Name []:kumu
Email Address [root@kumu]:
```

__生成 Server 端证书 Server Key:__

```
# ./build-key-server kumu_server # 一路回车，密码处填写密码
Generating a 1024 bit RSA private key
....................................++++++
.........++++++
writing new private key to 'kumu_server.key'
-----
You are about to be asked to enter information that will be incorporated
into your certificate request.
What you are about to enter is what is called a Distinguished Name or a DN.
There are quite a few fields but you can leave some blank
For some fields there will be a default value,
If you enter '.', the field will be left blank.
-----
Country Name (2 letter code) [CN]:
State or Province Name (full name) [BJ]:
Locality Name (eg, city) [BJ]:
Organization Name (eg, company) [kumu]:
Organizational Unit Name (eg, section) [kumu]:
Common Name (eg, your name or your server's hostname) [kumu_server]:
Name [kumu]:
Email Address [root@kumu]:

Please enter the following 'extra' attributes
to be sent with your certificate request
A challenge password []:123321  # 输入密码
An optional company name []:kumu
Using configuration from /etc/openvpn/openssl.cnf
Check that the request matches the signature
Signature ok
The Subject's Distinguished Name is as follows
countryName           :PRINTABLE:'CN'
stateOrProvinceName   :PRINTABLE:'BJ'
localityName          :PRINTABLE:'BJ'
organizationName      :PRINTABLE:'kumu'
organizationalUnitName:PRINTABLE:'kumu'
commonName            :T61STRING:'kumu_server'
name                  :PRINTABLE:'kumu'
emailAddress          :IA5STRING:'root@kumu'
Certificate is to be certified until May 11 22:54:08 2023 GMT (3650 days)
Sign the certificate? [y/n]:y


1 out of 1 certificate requests certified, commit? [y/n]y
Write out database with 1 new entries
Data Base Updated
```

__生成 Client 端证书:__

```
# ./build-key kumu_client1  # 一路回车，密码处填写密码
Generating a 1024 bit RSA private key
..++++++
.....................++++++
writing new private key to 'kumu_client1.key'
-----
You are about to be asked to enter information that will be incorporated
into your certificate request.
What you are about to enter is what is called a Distinguished Name or a DN.
There are quite a few fields but you can leave some blank
For some fields there will be a default value,
If you enter '.', the field will be left blank.
-----
Country Name (2 letter code) [CN]:
State or Province Name (full name) [BJ]:
Locality Name (eg, city) [BJ]:
Organization Name (eg, company) [kumu]:
Organizational Unit Name (eg, section) [kumu]:
Common Name (eg, your name or your server's hostname) [kumu_client1]:
Name [kumu]:
Email Address [root@kumu]:

Please enter the following 'extra' attributes
to be sent with your certificate request
A challenge password []:123321
An optional company name []:kumu
Using configuration from /etc/openvpn/openssl.cnf
Check that the request matches the signature
Signature ok
The Subject's Distinguished Name is as follows
countryName           :PRINTABLE:'CN'
stateOrProvinceName   :PRINTABLE:'BJ'
localityName          :PRINTABLE:'BJ'
organizationName      :PRINTABLE:'kumu'
organizationalUnitName:PRINTABLE:'kumu'
commonName            :T61STRING:'kumu_client1'
name                  :PRINTABLE:'kumu'
emailAddress          :IA5STRING:'root@kumu'
Certificate is to be certified until May 11 23:01:12 2023 GMT (3650 days)
Sign the certificate? [y/n]:y


1 out of 1 certificate requests certified, commit? [y/n]y
Write out database with 1 new entries
Data Base Updated
# ls /etc/openvpn/keys/
01.pem  ca.key      index.txt.attr      kumu_client1.crt  kumu_server.crt  openvpn-status.log
02.pem  dh1024.pem  index.txt.attr.old  kumu_client1.csr  kumu_server.csr  serial
ca.crt  index.txt   index.txt.old       kumu_client1.key  kumu_server.key  serial.old
```

__注__：生成其他客户端证书以此类推，名字不相同即可

__证书加密__:

```
# ./build-dh 
./build-dh: line 7: dhparam: command not found
```

出现如上问题，修改 `./build-dh` 命令中 `$OPENSSL` 为 `openssl` 即可，原因是默认 `/usr/local/src/openvpn/easy-rsa/easy-rsa/2.0/vars` 文件定义了 `OPENSSL=openssl` ,而笔者没有引用 `vars` 文件

```
# ./build-dh 
Generating DH parameters, 1024 bit long safe prime, generator 2
This is going to take a long time
... ...
# openvpn --genkey --secret /etc/openvpn/keys/ta.key  # 生成加密 key
```

### 2.2 Server 端配置文件修改

```
# pwd
/usr/local/src/openvpn/openvpn-2.3.1/sample/sample-config-files
# ls 
client.conf  loopback-client  openvpn-shutdown.sh  server.conf         tls-home.conf         xinetd-server-config
firewall.sh  loopback-server  openvpn-startup.sh   static-home.conf    tls-office.conf
home.up      office.up        README               static-office.conf  xinetd-client-config
# cp server.conf /etc/openvpn/  # 拷贝 Server 模板配置文件到配置目录
```

__Server 端配置文件内容如下:__

```
# grep -vE '^;|^$|^#' /etc/openvpn/server.conf 
port 1194
proto udp
dev tun
ca /etc/openvpn/keys/ca.crt
cert /etc/openvpn/keys/kumu_server.crt
key /etc/openvpn/keys/kumu_server.key  # This file should be kept secret
dh /etc/openvpn/keys/dh1024.pem
server 10.8.0.0 255.255.255.0
push "route 192.168.10.0 255.255.255.0" # 推送路由
client-to-client
keepalive 10 120
tls-auth /etc/openvpn/keys/ta.key 0 # This file is secret
comp-lzo
persist-key
persist-tun
status /etc/openvpn/keys/openvpn-status.log
verb 3
```

### 2.3 开启路由转发和启动 Openvpn

__开启路由转发:__

```
echo 1 > /proc/sys/net/ipv4/ip_forward # 临时开启
```

或者修改 /etc/sysctl.conf 中 `net.ipv4.ip_forward = 1`，执行 `sysctl -p` 永久生效

__启动服务:__

```
openvpn --config /etc/openvpn/server.conf --daemon
```

### 2.4 Windows 客户端连接配置

* 64 位安装 [openvpn-2.3.1-X86_64.exe](http://swupdate.openvpn.org/community/releases/openvpn-install-2.3.1-I001-x86_64.exe)
* 32 位请安装 [openvpn-2.3.1-i686.exe](http://swupdate.openvpn.org/community/releases/openvpn-install-2.3.1-I001-i686.exe)

拷贝 Server 端生成的如下客户端证书到 Windows 软件安装目录 `OpenVPN\config` 下

* kumu_client1.crt 
* kumu_client1.key 
* ca.key 
* ta.key

在 `OpenVPN\config` 目录中新建 Client 端配置文件 `client.ovpn`

```
client
dev tun
proto udp
remote 10.2.0.110 1194
resolv-retry infinite
nobind
persist-key
persist-tun
ca ca.crt
cert kumu_client1.crt
key kumu_client1.key
ns-cert-type server
tls-auth ta.key 1
comp-lzo
verb 3
```

Win7/Win8 以管理员身份启动 Openvpn Windows 客户端即可，基本的 Windows 安装这里不作介绍，如果正常，Openvpn Gui 客户端显示绿色，ping 测试无误，如下

```
C:\Users\kumu>ping 10.8.0.1  # 测试 VPN

正在 Ping 10.8.0.1 具有 32 字节的数据:
来自 10.8.0.1 的回复: 字节=32 时间<1ms TTL=64
来自 10.8.0.1 的回复: 字节=32 时间<1ms TTL=64
... ...
C:\Users\kumu>ping 192.168.10.19 #测试内网

正在 Ping 192.168.10.19 具有 32 字节的数据:
来自 192.168.10.19 的回复: 字节=32 时间=1ms TTL=64
来自 192.168.10.19 的回复: 字节=32 时间=2ms TTL=64
... ...
```

### 2.5 Linux 客户端配置

__安装__参见 Server 端安装

#### 相关配置

```
# mkdir /etc/openvpn
# cp /usr/local/src/openvpn/openvpn-2.3.1/sample/sample-config-files/client.conf /etc/openvpn/
# grep -vE '^$|^#|^;' /etc/openvpn/client.conf
client
dev tun
proto udp
remote 10.2.0.110 1194
resolv-retry infinite
nobind
persist-key
persist-tun
ca ca.crt
cert kumu_client1.crt
key kumu_client1.key
ns-cert-type server
tls-auth ta.key 1
comp-lzo
verb 3
```

拷贝 Server 端生成的如下客户端证书到Linux客户端 /etc/openvpn 下(这里为了方便不再生成一套客户端证书了)

* kumu_client1.crt
* kumu_client1.key
* ca.key
* ta.key 

__启动 Openvpn 客户端服务:__

```
openvpn --config /etc/openvpn/client.conf --daemon
```

#### 测试

```
# ifconfig tun0
tun0      Link encap:UNSPEC  HWaddr 00-00-00-00-00-00-00-00-00-00-00-00-00-00-00-00  
          inet addr:10.8.0.6  P-t-P:10.8.0.5  Mask:255.255.255.255
... ...
# route -n
Kernel IP routing table
Destination     Gateway         Genmask         Flags Metric Ref    Use Iface
10.8.0.5        0.0.0.0         255.255.255.255 UH    0      0        0 tun0
10.8.0.0        10.8.0.5        255.255.255.0   UG    0      0        0 tun0
192.168.10.0    10.8.0.5        255.255.255.0   UG    0      0        0 tun0
... ...
# ping 192.168.10.19
PING 192.168.10.19 (192.168.10.19) 56(84) bytes of data.
64 bytes from 192.168.10.19: icmp_seq=1 ttl=64 time=0.690 ms
64 bytes from 192.168.10.19: icmp_seq=2 ttl=64 time=1.21 ms
... ...
```


