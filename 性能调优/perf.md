
# 1. 什么是 Perf #

Perf is a profiler tool for Linux 2.6+ based systems that abstracts away CPU
hardware differences in Linux performance measurements and presents a simple
commandline interface. Perf is based on the perf_events interface exported by
recent versions of the Linux kernel. 

The perf tool offers a rich set of commands to collect and analyze performance
and trace data.

	usage: perf [--version] [--help] COMMAND [ARGS]
	
	The most commonly used perf commands are:
	  annotate        Read perf.data (created by perf record) and display annotated code
	  archive         Create archive with object files with build-ids found in perf.data file
	  bench           General framework for benchmark suites
	  buildid-cache   Manage build-id cache.
	  buildid-list    List the buildids in a perf.data file
	  diff            Read two perf.data files and display the differential profile
	  evlist          List the event names in a perf.data file
	  inject          Filter to augment the events stream with additional information
	  kmem            Tool to trace/measure kernel memory(slab) properties
	  kvm             Tool to trace/measure kvm guest os
	  list            List all symbolic event types
	  lock            Analyze lock events
	  record          Run a command and record its profile into perf.data
	  report          Read perf.data (created by perf record) and display the profile
	  sched           Tool to trace/measure scheduler properties (latencies)
	  script          Read perf.data (created by perf record) and display trace output
	  stat            Run a command and gather performance counter statistics
	  test            Runs sanity tests.
	  timechart       Tool to visualize total system behavior during a workload
	  top             System profiling tool.
	  trace           strace inspired tool
	  probe           Define new dynamic tracepoints
	
	See 'perf help COMMAND' for more information on a specific command.
	

# 2. Perf 的功能 #

+ 评估程序对硬件资源的使用情况: 各级 Cache 访问次数、各级 Cache 丢失次数、流水线
停顿周期、前端总线访问次数 … …
+ 评估程序对操作系统资源的使用情况: 系统调用次数、Page Fault 次数、上下文切换次
数、任务迁移次数 … …
+ 评估内核性能: Benchmarks、调度器性能分析、系统行为记录与重演动态添加探测点 … …



## 2.1 Perf list ##


功能: 查看当前软硬件环境、支持的性能事件。

查看所有分类事件的个数: `perf list | awk -F: '/Tracepoint event/ { lib[$1]++ } END { for (l in lib) { printf "  %-16s %d\n", l, lib[l] } }' | sort | column`

These include:

+ block: block device I/O
+ ext3, ext4: file system operations
+ kmem: kernel memory allocation events
+ random: kernel random number generator events
+ sched: CPU scheduler events
+ syscalls: system call enter and exits
+ task: task events


性能事件分三类:

1. 硬件性能事件
2. 软件性能事件
3. tracepoint event


## 2.2 Perf stat ##

功能: 分析程序的整体性能.

示例:

1. `perf stat -e task-clock ./a.out`  # 分析 task-clock 事件
2. `perf stat -p 31317`               # 分析 31317 进程
3. `perf stat -t 31318`              # 分析 31318 线程
4. `perf stat -r 5 ./a.out`          # 分析 5 次就停止
5. `perf stat -d ./a.out`            # 全面分析

## 2.3 Perf top ##

功能: 实时显示系统/进程的性能统计信息。

使用方法: `perf top <options>` 分析界面之后 `?` 可以查看快捷键。有用的是 `a`，
即: "annotate current symbol", 可以看到具体的指令。

示例:

1. `perf top -a -p 31317` # 分析整个 31317 进程的性能
2. `perf top -n -p 31317` # 显示事件数量
3. `perf top --show-total-period -p 31317` # 累计事件个数
4. `perf top -G`

top 界面解释:

+ 第一列: 性能事件在整个检测域中占的比例，符号的 热度
+ 第二列: 符号所在 DSO(Dynamic Shared Object)
+ 第三列: DSO 类型(ELF可执行文件，动态链接库，内核，内核模块，VDSO 等)
+ 第四列: 符号名(函数名)


## 2.4 Perf record/report ##

功能:

1. record: 记录一段时间内系统/进程的性能事件，生成 perf.data 文件
2. report: 读取 perf.data 文件，并显示分析数据

使用: `perf record [<options>] [<command>]`

示例:

	perf record -g -p 31655  # 记录一段时间的 zone_server 性能事件
	perf report           # 得到分析结果
	perf report -n        # 显示对应时间的调用次数
	perf report -v        # 显示每个符号的地址
	perf report -g flat,5%    #
	perf report -g graph,5%   #
	perf report -g fractal,5% #


## 2.5 Perf timechart ##

功能: 将系统的运行状态以 SVG 图的形式输出。

1. 各处理器状态(run, idle)
2. 各进程的时间图谱(run, sleep, blocked ...)

示例:

    perf timechart record -p 31655

## 2.6 Perf script ##

功能: 查看 perf 数据文件 (perf.data)

示例:

+ `perf script -l` # 查看当前系统中可扩展脚本
+ `perf script syscall-count` # 查看系统调用被调度次数
+ `perf script sctop` # 实时查看各个系统调用被调用的次数

## 2.7 Perf kmem ##

功能: 针对内存子系统的分析工具。也可以用 `perf record -e kmem:* -p 630` 来统计内存分配、释放。

## 2.8 内核 tracepoint ##

暂无.

# 3. 一些术语 ##

**PMU** -- Performance Monitoring Unit，性能检测单元。

**Tracepoints** -- Tracepoint 是散落在内核源代码中的一些 hook，一旦使用，它们便可以在
特定的代码被运行到时被触发，这一特性可以被各种 trace/debug 工具所使用。

**page-fault** -- Linux的内存管理子系统采用了分页机制。当应用程序请求的页面尚未建立、
请求的页面不在内存中、请求的页面虽然在内存中，但尚未建立物理地址与虚拟 地址的映
射关系时，都会触发一次缺页异常(page-fault)。

**task-clock** -- 应用程序真正占用处理器的时间，单位是毫秒。任务执行时间。Linux是多任
务分时操作系统中，一个任务不太可能在执行期间始终占据处理器。操作系统会根据调度策
略合理安排各个任务轮流使用处理器，每次调度会产生一次上下文切换。

**cycle** -- 程序消耗的处理器周期数。

**instructions** -- 程序执行期间产生的处理器指令数。

**branches** -- 程序在执行期间遇到的分支指令数。绝大多数现在处理器都具有分支预测与OOO
(Out-of-Order)执行机制，以充分利用 CPU 内部的资源，减少流水线停顿周期。当处理器遇
到分支指令时，正常来说，需要等待分支条件计算完毕才能知道后续指令该往何处跳转。这
就导致了在等待分支条件计算期间，流水线上出现若干周期的停顿(流水线Hazard)。


# 4. 参考(扩展)资料 #

+ [Perf 在 Linux 程序性能评估中的应用(by淘宝承刚)]( http://kernel.taobao.org/images/5/52/Perf%E5%9C%A8Linux%E6%80%A7%E8%83%BD%E8%AF%84%E4%BC%B0%E4%B8%AD%E7%9A%84%E5%BA%94%E7%94%A8_v3.pdf)
+ [Linux 的系统级性能剖析工具‐perf(二)](http://kernel.taobao.org/images/3/31/Linux%E7%9A%84%E7%B3%BB%E7%BB%9F%E7%BA%A7%E6%80%A7%E8%83%BD%E5%89%96%E6%9E%90%E5%B7%A5%E5%85%B7-perf-2.pdf)
+ [Linux 的系统级性能剖析工具‐perf(三)](http://kernel.taobao.org/images/e/e4/Linux%E7%9A%84%E7%B3%BB%E7%BB%9F%E7%BA%A7%E6%80%A7%E8%83%BD%E5%89%96%E6%9E%90%E5%B7%A5%E5%85%B7-perf-3.pdf)
+ [Linux kernel profiling with perf](https://perf.wiki.kernel.org/index.php/Tutorial)
+ [Perf -- Linux下的系统性能调优工具，第 1 部分](http://www.ibm.com/developerworks/cn/linux/l-cn-perf1/)
+ [Perf -- Linux下的系统性能调优工具，第 2 部分](http://www.ibm.com/developerworks/cn/linux/l-cn-perf2/)
+ [perf Examples](http://www.brendangregg.com/perf.html)

