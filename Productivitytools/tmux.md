# tmux

对于终端复用工具这里推荐使用tmux，当然此类工具比较好的还有screen，不过相对[screen](http://www.ibm.com/developerworks/cn/linux/l-cn-screen/) 这里我更倾向于推荐tmux[强悍的分屏等]。
  
如果仅仅只是多标签的功能，那么putty结合一些工具也可以做到，或者干脆使用xshell，当然tmux此类工具不仅仅是那么简单。另外一个选择使用tmux/screen工具的原因是，有时我们会经常需要SSH或者telent远程登录到Linux服务器，有一些任务需要长时间运行，比如系统备份、数据传输等。通常情况下我们都是开一个远程终端窗口，因为执行时间比较长，一般需要等待它执行完毕，在此过程中不能关闭窗口或者网络原因断开连接，断开之后就Game Over了。这个功能就有点类似`nohup`和`setsid`命令的实现了，而`tmux/screen`则集`nohup/setsid`和多标签于一身。

## 安装

安装的话这里就不过说明了，不同的Linux发行版相应的包管理机制不同，安装tmux包即可。

## 使用实例

### 几个名词

tmux主要包括以下几个模块：

*   session
    *   会话:一个服务器可以包含多个会话
*   window
    *   窗口:一个会话可以包含多个窗口
*   pane
    *   面板:一个窗口可以包含多个面板[强悍的分屏]

### 绑定快捷键

列出了tmux的几个基本模块之后，就要来点实实在在的干货了，和`screen`默认激活控制台的`Ctrl+a`不同，tmux默认的是`Ctrl+b`，使用快捷键之后就可以执行一些相应的指令了。当然如果你不习惯使用`Ctrl+b`，也可以在`~/.tmux`文件中加入以下内容把快捷键变为`Ctrl+a`：

``` bash
# Set prefix key to Ctrl-a
unbind-key C-b
set-option -g prefix C-a
```

以下所有的操作都是激活控制台之后，即键入`Ctrl+b`前提下才可以使用的命令【这里假设快捷键没改，改了的话则用`Ctrl+b`】。

### 基本操作

|?      | 列出所有快捷键；按q返回                                                          |
|:----- | :------------------------------------------------------------------------------- |
|d      | 脱离当前会话,可暂时返回Shell界面，输入tmux attach能够重新进入之前会话            |
|s      | 选择并切换会话；在同时开启了多个会话时使用                                       |
|D      | 选择要脱离的会话；在同时开启了多个会话时使用                                     |
|:      | 进入命令行模式；此时可输入支持的命令，例如kill-server所有tmux会话                |
|[      | 复制模式，光标移动到复制内容位置，空格键开始，方向键选择复制，回车确认，q/Esc退出|
|]      | 进入粘贴模式，粘贴之前复制的内容，按q/Esc退出                                    |
|~      | 列出提示信息缓存；其中包含了之前tmux返回的各种提示信息                           |
|t      | 显示当前的时间                                                                   |
|Ctrl+z | 挂起当前会话                                                                     |


  
### 窗口操作
    
|   c           |创建新窗口                               |
| :-----------  | :-------------------------------------  |
|   &           |关闭当前窗口                             |
|   数字键      |切换到指定窗口                           |
|   p           |切换至上一窗口                           |
|   n           |切换至下一窗口                           |
|   l           |前后窗口间互相切换                       |
|   w           |通过窗口列表切换窗口                     |
|   ,           |重命名当前窗口，便于识别                 |
|   .           |修改当前窗口编号，相当于重新排序         |
|   f           |在所有窗口中查找关键词，便于窗口多了切换 |

### 面板操作:
    
|   "               |   将当前面板上下分屏                                      |
|:--------------    |:--------------------------------------------------------  |
|   %               |   将当前面板左右分屏                                      |
|   x               |   关闭当前分屏                                            |
|   !               |   将当前面板置于新窗口,即新建一个窗口,其中仅包含当前面板  |   
|   Ctrl+方向键     |   以1个单元格为单位移动边缘以调整当前面板大小             |
|   Alt+方向键      |   以5个单元格为单位移动边缘以调整当前面板大小             |
|   空格键          |   可以在默认面板布局中切换，试试就知道了                  |
|   q               |   显示面板编号                                            |
|   o               |   选择当前窗口中下一个面板                                |
|   方向键          |   移动光标选择对应面板                                    |
|   {               |   向前置换当前面板                                        |
|   }               |   向后置换当前面板                                        |
|   Alt+o           |   逆时针旋转当前窗口的面板                                |
|   Ctrl+o          |   顺时针旋转当前窗口的面板                                |
|   z               |   tmux 1.8新特性，最大化当前所在面板                      |
  
## .tmux.conf基本配置

软件到手了，自己怎么舒服就怎么用。定制主要还是在于`.tmux.conf`配置文件的配置，以下列出我的配置文件：

``` bash
# Set prefix key to Ctrl-a
unbind-key C-b
set-option -g prefix C-a
bind-key C-a last-window # 方便切换，个人习惯
bind-key a send-prefix   
# shell下的Ctrl+a切换到行首在此配置下失效，此处设置之后Ctrl+a再按a即可切换至shell行首
 
# reload settings   # 重新读取加载配置文件
bind R source-file ~/.tmux.conf \; display-message "Config reloaded..."

# Ctrl-Left/Right cycles thru windows (no prefix) 
# 不使用prefix键，使用Ctrl和左右方向键方便切换窗口
bind-key -n "C-Left" select-window -t :-
bind-key -n "C-Right" select-window -t :+

# displays 
bind-key * list-clients

set -g default-terminal "screen-256color"   # use 256 colors
set -g display-time 5000                    # status line messages display
set -g status-utf8 on                       # enable utf-8 
set -g history-limit 100000                 # scrollback buffer n lines
setw -g mode-keys vi                        # use vi mode

# start window indexing at one instead of zero 使窗口从1开始，默认从0开始 
set -g base-index 1 

# key bindings for horizontal and vertical panes
unbind %
bind | split-window -h      # 使用|竖屏，方便分屏
unbind '"' 
bind - split-window -v      # 使用-横屏，方便分屏

# window title string (uses statusbar variables)
set -g set-titles-string '#T'

# status bar with load and time 
set -g status-bg blue 
set -g status-fg '#bbbbbb'
set -g status-left-fg green 
set -g status-left-bg blue
set -g status-right-fg green
set -g status-right-bg blue 
set -g status-left-length 90 
set -g status-right-length 90
set -g status-left '[#(whoami)]'
set -g status-right '[#(date +" %m-%d %H:%M ")]'
set -g status-justify "centre"
set -g window-status-format '#I #W'
set -g window-status-current-format ' #I #W '
setw -g window-status-current-bg blue
setw -g window-status-current-fg green

# pane border colors
set -g pane-active-border-fg '#55ff55'
set -g pane-border-fg '#555555'
```

注：关于256 colors需要设置相应别名`export ssh='TERM=xterm ssh'`

### 开启批量执行
  
如果已经修改prefix键位`Ctrl+a`，则`Ctrl+a`[默认Ctrl+b]后输入`:set synchronize-panes` ，输入:set sync [TAB]键可自动补齐
  
取消批量执行模式重复之前操作即可

## 脚本化启动
  
把以下脚本内容加入到`~/.bashrc`，即可每次登录进入到tmux

``` bash
tmux_init()
{
    tmux new-session -s "kumu" -d -n "local"    # 开启一个会话
    tmux new-window -n "other"          # 开启一个窗口
    tmux split-window -h                # 开启一个竖屏
    tmux split-window -v "top"          # 开启一个横屏,并执行top命令
    tmux -2 attach-session -d           # tmux -2强制启用256color，连接已开启的tmux
}

# 判断是否已有开启的tmux会话，没有则开启
if which tmux 2>&1 >/dev/null; then
    test -z "$TMUX" && (tmux attach || tmux_init)
fi
```

## 参考文档
  
* [使用tmux](https://wiki.freebsdchina.org/software/t/tmux)
* [archlinux tmux](https://wiki.archlinux.org/index.php/Tmux)
* [Tankywoo tmux wiki](http://wiki.wutianqi.com/software/tmux.html)
* [Tmux 使用心得小记 ](http://www.lovelin.info/blog/2012/10/25/tmuxshi-yong-xin-de-xiao-ji/)
* [Tmux Openbsd Manual Pages](http://www.openbsd.org/cgi-bin/man.cgi?query=tmux&sektion=1)