## 一、Cron 基础

### 1.1 什么是 cron, crond, crontab

> **cron** is the general name for the service that runs scheduled actions. **crond** is the name of the daemon that runs in the background and reads **crontab** files. 

简单理解：cron 是服务，crond 是守护进程， crontab 的 crond 的配置文件。

### 1.2 crontab 选项

+ `crontab -e` : Edit your crontab file, or create one if it doesn't already exist. # 推荐使用命令新增计划任务--语法检查
+ `crontab -l` : Display your crontab file.
+ `crontab -r` : Remove your crontab file. # 慎用
+ `crontab -u user` : Used in conjunction with other options, this option allows you to modify or view the crontab file of user. When available, only administrators can use this option.

### 1.3 crontab 格式

    minute(s) hour(s) day(s) month(s) weekday(s) command(s)

``` 
# Use the hash sign to prefix a comment
# +—————- minute (0 – 59)
# |  +————- hour (0 – 23)
# |  |  +———- day of month (1 – 31)
# |  |  |  +——- month (1 – 12)
# |  |  |  |  +—- day of week (0 – 7) (Sunday=0 or 7)
# |  |  |  |  |
# *  *  *  *  *  command to be executed
```

## 二、使用举例

使用命令 `crontab -e` 编辑 crontab 文件。

(1) 在每天的 7 点同步服务器时间

    0 7 * * * ntpdate 192.168.1.112


(2) 每两个小时执行一次

    0 */2 * * * echo "2 minutes later" >> /tmp/output.txt

(3) 每周五早上十点写周报

    0 10 * * * 5 /home/jerryzhang/update_weekly.py

(4) 每天 6, 12, 18 点执行一次命令

    0 6,12,18 * * *  /bin/echo hello

(5) 每天 13, 14, 15, 16, 17 点执行一次命令

    0 13-17 * * *  /bin/echo hello

__注:__

* 程序执行完毕，系统会给对应用户发送邮件，显示该程序执行内容，如果不想收到，可以重定向内容 `> /dev/null 2>&1`
* 如果执行语句中有 `%` 号，需要使用反斜杠 '\' 转义

## 三、参考资料

+ [维基百科: Cron](http://zh.wikipedia.org/wiki/Cron)
+ [Difference between Cron and Crontab?](http://stackoverflow.com/questions/21789148/difference-between-cron-and-crontab)
+ [Cron and Crontab usage and examples](http://www.pantz.org/software/cron/croninfo.html)
+ [What are cron and crontab, and how do I use them?](https://kb.iu.edu/d/afiz)
+ [crontab实例+详解](http://88263188.blog.51cto.com/416252/291683)
