# top

`top`命令是Linux下常用的性能分析工具，能够实时显示系统中各个进程的资源占用状况，类似于Windows的任务管理器。
 
## 一、top理论
 
``` bash
# top
top - 10:52:40 up 3 days, 52 min,  1 user,  load average: 57.28, 112.40, 123.60
Tasks:  99 total,   1 running,  98 sleeping,   0 stopped,   0 zombie
Cpu(s): 19.5%us, 11.4%sy,  0.0%ni,  0.0%id, 65.7%wa,  0.0%hi,  3.4%si,  0.0%st
Mem:  16435896k total, 16232468k used,   203428k free,    58004k buffers
Swap:  1044476k total,   713552k used,   330924k free, 10052032k cached
 
  PID USER      PR  NI  VIRT  RES  SHR S %CPU %MEM    TIME+  COMMAND
  ... ...
```
 
统计信息区前五行是系统整体的统计信息。第一行是任务队列信息，同`uptime`命令的执行结果。其内容如下：
 
|内容              | 解释
|:---              |:---   
|10:52:40          | 当前时间          
|up 3 days, 52 min | 系统运行时间
|1 users           | 当前登录用户数
|load average: 57.28, 112.40, 123.60 | 系统负载，即任务队列平均长度。分别为1、5、15min前到现在平均值。
 
第二、三行为进程和CPU的信息。当有多个CPU时，这些内容可能会超过两行。内容如下：

| 内容                                | 解释
|:---                                 |:---
| Tasks:99 total                      | 进程总数[键入H可查看线程数]
| 1 running,  98 sleeping,  0 stopped | 正在运行的进程、睡眠进程、停止的进程
| 0 zombie                            | 僵尸进程数</td>
| Cpu(s): 19.5%us, 11.4%sy,           | 用户空间占用CPU百分比、内核空间占用CPU百分比
| 0.0%ni, 0.0%id,                     | 用户进程空间内改变进程优先级占用CPU、空闲CPU百分比
| 65.7%wa, 0.0%hi, 3.4%si, 0.0%st     | 等待IO的CPU时间百分比，最后三个是中断请求相关
 
倒数第2、3行为内存相关信息：

|内容                                  | 解释
|:---                                  |:---
|Mem: 16435896k total, 16232468k used, | 分别是物理内存总量、使用物理内存总量
|:---                                  |:---
|203428k free, 58004k buffers          | 空闲内存总量、用作内核缓存内存量
|Swap: 1044476k total, 713552k used,   | 分别是交换分区量、使用交换分区总量
|330924k free, 10052032k cached        | 空闲交换区总量、缓存交换区总量
 
 
- buffe   [Difference between buffer and cache](http://wiki.answers.com/Q/Difference_between_buffer_and_cache)
 
>A data area, shared by hardware devices or program a process is called buffer. They are operated at different speeds or with different sets of priorities. The buffer allows each device or process to operate without holding up by the other. In order to a buffer to be effective, the size of the buffer needs to be considered by the buffer designer. Like a cache, a buffer is a "midpoint holding place" but does not exist so much to accelerate the speed of an activity as for supporting the coordination of separate activities.
 
>This term is used not only in programming but in hardware as well. In programming, buffering sometimes needs to screen data from its final intended place so that it can be edited or processed before moving to a regular file or database.
 
- cached
 
>Cache memory is type of random access memory (RAM). Cache Memory can be accessed more quickly by the computer microprocessor than it can be accessed by regular RAM. Like microprocessor processes data, it looks first in the cache memory and if there, it finds the data from a previous reading of data, it does not need to do the more time consuming reading of data from larger memory. 
 
>Sometimes Cache memory is described in levels of closeness and convenience to the microprocessor. An L1 cache is on the same chip like the microprocessors.
 
>In addition to cache memory, RAM itself is a cache memory for hard disk storage since all of RAM's contents come up to the hard disk initially when you turn on your computer and load the operating system that you are loading it into RAM and later when you start new applications and access new data. RAM also contains a special area called a disk cache that consists of the data most recently read in from the hard disk.
 
最后1行则是进程相关的资源占用信息:
 
- `PID`：进程的ID
- `USER`：进程所有者
- `PR`：进程的优先级别，越小越优先被执行
- `NI`：nice值。负值表示高优先级，正值表示低优先级
- `VIRT`：进程占用的虚拟内存
- `RES`：进程占用的物理内存
- `SHR`：进程使用的共享内存
- `S`：进程的状态。S表示休眠，R表示正在运行，Z表示僵死状态，N表示该进程优先值为负数
- `%CPU`：进程占用CPU的使用率
- `%MEM`：进程使用的物理内存和总内存的百分比
- `TIME+`：该进程启动后占用的总的CPU时间，即占用CPU使用时间的累加值。
- `COMMAND`：进程启动命令名称
 
## 二、top技巧
 
终端执行top命令之后【也可后接一些选项，比如`top -p 1`只监控init进程，`top -u root`只显示root运行进程等等】，可以敲击如下按键，实现不同功能：
 
- `h`：获取top的命令帮助
- `1`(数字1)：列出所有的单个CPU负载情况
- `z`：top显示颜色
    - `x`：类似高亮显示，在`z`模式下使用
- `P`[大写]：按CPU占用高低顺序列出程序
- `M`[大写]：按内存占用高低顺序列出程序
- `c`：显示进程命令的全路径与参数
- `H`：显示线程，默认只显示进程
- top默认按cpu占用排序，按F(大写)即可选择相应排序
- `d`：top默认刷新时间是3s，使用d键可自定义刷新时间 
- top类似上下翻页的方法：
    - shift+<：下翻页
    - shift+>：上翻页
- `f`：可以指定top显示的内容，如ppid、swap等都可以选择显示
    - 显示`Swap`利用率：按`f`键，然后按`p`键，回车即可看到Swap状态 
- `k`：输入k之后可以kill掉指定的进程
- `A`：分类显示各种系统资源高的进程。可用于快速识别系统上的性能要求极高的任务，__推荐使用__
- `W`[大写]:将当前设置写入`~/.toprc`文件中。这是写top配置文件的推荐方法
 
## 三、参考

- [鸟哥Linux私房菜](http://linux.vbird.org/) 
- [top - Process Activity Command](http://www.cyberciti.biz/tips/top-linux-monitoring-tools.html)
- [Learning Linux Commands: top](http://how-to.linuxcareer.com/learning-linux-commands-top)
