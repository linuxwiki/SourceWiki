# DNS 查询工具 -- dig|host|nslookup

`nslookup`、`host`和`dig` 是三个DNS查询工具，以下会分别介绍它们的使用方法。

## 一、nslookup

>nslookup is a tried and true program that has weathered the ages. nslookup has been deprecated and may be removed from future releases. There is not even a man page for this program.

因此，这里不过多介绍。

## 二、host

`host`命令和`dig`命令很相像，但是`host`命令的输出要更简洁，如下示例

``` bash
# host www.google.com
www.google.com has address 74.125.135.106
```

`host`命令只输出给我们`dig`命令的ANSWER section，相对`dig`提供的一些不必要的信息来说更简洁快速。也可指定DNS Server来查询，例如我想使用Google DNS`8.8.8.8`,named可以如下指定

``` bash
# host www.google.com 8.8.8.8
Using domain server:
Name: 8.8.8.8
Address: 8.8.8.8#53
Aliases: 

www.google.com has address 173.194.72.147
```

`host`当然也支持反解析

``` bash
# host 173.194.72.147
147.72.194.173.in-addr.arpa domain name pointer tf-in-f147.1e100.net.
```

指定查询类型可以使用`-t`选项

``` bash
# host -t SOA google.com  #查询SOA记录信息
google.com has SOA record ns1.google.com. dns-admin.google.com. 2013061100 7200 1800 1209600 300
```

查询`MX`记录

```
$ host -t MX google.com 
google.com mail is handled by 10 aspmx.l.google.com.
google.com mail is handled by 40 alt3.aspmx.l.google.com.
google.com mail is handled by 20 alt1.aspmx.l.google.com.
google.com mail is handled by 50 alt4.aspmx.l.google.com.
google.com mail is handled by 30 alt2.aspmx.l.google.com.
```

`-C`对比认证DNS SOA信息

```
# host -C google.com
Nameserver 216.239.34.10:
        google.com has SOA record ns1.google.com. dns-admin.google.com. 2013061100 7200 1800 1209600 300
Nameserver 216.239.36.10:
        google.com has SOA record ns1.google.com. dns-admin.google.com. 2013061100 7200 1800 1209600 300
Nameserver 216.239.32.10:
... ...
```

查询DNS Server软件版本信息,10.10.10.2为DNS Server

``` bash
# host -c CH -t txt version.bind 10.10.10.2  
Using domain server:
Name: 10.10.10.2
Address: 10.10.10.2#53
Aliases: 

version.bind descriptive text "9.8.1-P2"
```

__host帮助__

``` bash
# host
Usage: host [-aCdlriTwv] [-c class] [-N ndots] [-t type] [-W time]
            [-R number] [-m flag] hostname [server]
       -a is equivalent to -v -t ANY
       -c specifies query class for non-IN data  搜索非网络数据时要指定要查找的类
       -C compares SOA records on authoritative nameservers
       -d is equivalent to -v
       -l lists all hosts in a domain, using AXFR
       -i IP6.INT reverse lookups
       -N changes the number of dots allowed before root lookup is done
       -r disables recursive processing
       -R specifies number of retries for UDP packets
       -s a SERVFAIL response should stop query
       -t specifies the query type 指定要查询的记录类型
       -T enables TCP/IP mode
       -v enables verbose output  输出更详细的信息
       -w specifies to wait forever for a reply
       -W specifies how long to wait for a reply
       -4 use IPv4 query transport only
       -6 use IPv6 query transport only
       -m set memory debugging flag (trace|record|usage)
```

## 三、dig

dig也是一个很强大的命令，相对host来说输出较为繁杂，如下：

``` bash
$ dig www.google.com
... ...

;; ANSWER SECTION:
www.google.com.         297     IN      A       74.125.135.106
www.google.com.         297     IN      A       74.125.135.104
... ...

;; AUTHORITY SECTION:
google.com.             172796  IN      NS      ns3.google.com.
google.com.             172796  IN      NS      ns1.google.com.
google.com.             172796  IN      NS      ns4.google.com.
google.com.             172796  IN      NS      ns2.google.com.

... ...
```

查询`MX`记录

``` bash
$ dig google.com MX | grep '^;; ANSWER SECTION:' -A 5
;; ANSWER SECTION:
google.com.             368     IN      MX      50 alt4.aspmx.l.google.com.
google.com.             368     IN      MX      40 alt3.aspmx.l.google.com.
google.com.             368     IN      MX      10 aspmx.l.google.com.
google.com.             368     IN      MX      30 alt2.aspmx.l.google.com.
google.com.             368     IN      MX      20 alt1.aspmx.l.google.com.
```

查询`SOA`记录

``` bash
$ dig google.com SOA | grep '^;; ANSWER SECTION:' -A 1
;; ANSWER SECTION:
google.com.             85539   IN      SOA     ns1.google.com. dns-admin.google.com. 2013061100 7200 1800 1209600 300
```

指定DNS Server查询

``` bash
$ dig www.baidu.com @8.8.8.8
... ...
;; ANSWER SECTION:
www.baidu.com.          1024    IN      CNAME   www.a.shifen.com.
www.a.shifen.com.       166     IN      A       119.75.217.56
www.a.shifen.com.       166     IN      A       119.75.218.77
... ...
```

`dig`查询版本号

``` bash
$ dig chaos txt version.bind  10.10.10.2 | grep '^;; ANSWER SECTION:' -A 1
;; ANSWER SECTION:
version.bind.           0       CH      TXT     "9.8.1-P2"
```

`dig`反解析`-x`

``` bash
$ dig -x 74.125.135.105
;; QUESTION SECTION:
;105.135.125.74.in-addr.arpa.   IN      PTR

;; ANSWER SECTION:
105.135.125.74.in-addr.arpa. 83205 IN   PTR     ni-in-f105.1e100.net.
```
