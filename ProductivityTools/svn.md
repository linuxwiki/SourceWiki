# 版本管理工具: SVN

本文由 [svn book 1.4](http://svndoc.iusesvn.com/svnbook/1.4) 整理而来。

`svn help <SUBCOMMAND>`: 查看子命令用法、参数及行为方式。

## 一、创建一个svn仓库

+ `svnadmin create /user/local/svn/newrepos`: 创建一个新的 repo
+ `svn import YourDir file:///usr/local/svn/newrepos/some/project -m "Init import"` : 把未版本化的文件导入版本库
+ `svn list file:///usr/local/svn/newrepos/some/project`: 查看文件列表
+ `svn export http://svn.example.com/svn/repos1` : 导出一份干净的仓库(没有.svn文件)

## 二、基本使用

+ `svn checkout <repo-url>` : 检出到本地
+ `svn update` : 更新工作拷贝
+ `svn add foo` : 将文件、目录或者符号链添加到版本库(提交后生效)
+ `svn delete foo` : 将文件、目录或者符号链从版本库中移除(提交后生效)
+ `svn copy foo bar` : 拷贝
+ `svn move foo bar` : 重命名
+ `svn mkdir blort` : 等价于 `mkdir blort; svn add blort`
+ `svn status` : 浏览你所做的修改。`-v` 显示所有项目文件的状态；`-u` 显示是否过期。
    - `A item` : 预定加入到版本库的文件、目录或符号链的item
    - `C item` : 文件 item 发生冲突，在从服务器更新时与本地版本发生交跌，在你提交到版本库前，必须手工的解决冲突
    - `D item` : 文件、目录或是符号链 item 预定从版本库中删除
    - `M item` : 文件 item 的内容被修改了
+ `svn diff` : 查看详细的信息，`-r` 指定比较版本
+ `svn revert` : 取消本地修改
+ `svn resolved` : 解决完冲突后通知svn(当 update 有冲突时会生成 3 个临时文件: filename.mine, filename.rOLDREV, filename.rNEWREV，resolved 告诉 svn 删除那3个临时文件)
+ `svn commmit` : 提交修改, `-m` 添加描述修改信息
+ `svn log` : 查看日志，`-r` 显示某一个版本, `-v` 详细模式
+ `svn cat` : 取特定版本的某一个文件显示在当前屏幕，`-r` 显示指定版本
+ `svn list` : 显示一个目录在某一版本存在的文件
+ `svn cleanup` : 它查找工作拷贝中的所有遗留的日志文件，删除进程中工作拷贝的锁

## 三、高级使用

### 3.1 属性

属性不是版本化的，如果你修改，删除一个修订版本属性，SVN 没有办法恢复到以前的值。

+ `svn propset` : 添加和修改文件或目录的属性
+ `svn propedit` : 使用定制的编辑器程序来添加和修改属性
+ `svn proplist` : 列出路径上存在的所有属性名称
+ `svn propget` : 获取属性的值
+ `svn propdel` : 删除某个属性

### 3.2 忽略未版本控制的条目

SVN 忽略方法和 git 不同，git 是在本地加一个 `.gitigore` 即可，SVN 是通过属性(svn:ignore)来实现的。

`svn propedit svn:ignore work_dir` : 在工作目录 work_dir 下添加过滤文件

### 3.3 外部定义

引用场景：多个目录共享同一个目录。以游戏开发为例，UI、服务器、策划共享的数据表，这个时候就可以用到 *外部定义* 这个概念了。我理解外部定义就相当于 linux 下的软链接。

一个外部定义是一个本地路经到 URL 的影射—也有可能一个特定的修订版本—一些版本化的资源。在 SVN 你可以使用 `svn:externals` 属性来定义外部定义，你可以用 `svn propset` 或` svn propedit` 创建和修改这个属性。它可以设置到任何版本化的路经，它的值是一个多行的子目录，可选的修订版本标记和完全有效的 SVN 版本库 URL 的列表（相对于设置属性的版本化目录）。

可以使用 `--ignore-externals` 来忽略 externals。


## 四、分支与合并

### 4.1 分支创建与删除

SVN 分支其实只是做了一份拷贝而已(svn copy), 但是它并不是物理意义上的拷贝(完全复制一份，可以理解成 Linux 的硬链接, *廉价复制* )。

    svn copy http://svn.example.com/repos/calc/trunk \
    http://svn.example.com/repos/calc/branches/my-calc-branch \
          -m "Creating a private branch of /calc/trunk."

+ `svn delete http://svn.example.com/repos/calc/branches/my-calc-branch  -m "Removing obsolete branch of calc project."` : 删除一个分支

### 4.2 在分支上工作

再 `checkout` 一份呗。

### 4.3 合并分支

    svn diff -c 344 http://svn.example.com/repos/calc/trunk
    svn merge -c 344 http://svn.example.com/repos/calc/trunk

合并分支看似简单，其实是一件非常头疼的事情(尤其是大项目)。我宁愿去选择一些可视化的文本比较工作，比如 [BeyondCompare](http://www.scootersoftware.com/)，可以为你省下太多事情了。

[版本库管理](http://svndoc.iusesvn.com/svnbook/1.4/svn.reposadmin.html) 之后的章节感兴趣去看看(上面提供的命令足够应付大多数使用情况了)

## 五、知识点

### 5.1 四种文件状态(svn status)

1. 未修改且是当前的, 文件在工作目录里没有修改，在工作修订版本之后没有修改提交到版本库。svn commit操作不做任何事情，svn update不做任何事情。
2. 本地已修改且是当前的, 在工作目录已经修改，从基本修订版本之后没有修改提交到版本库。本地修改没有提交，因此svn commit会成功提交，svn update不做任何事情。
3. 未修改且不是当前的了, 这个文件在工作目录没有修改，但在版本库中已经修改了。这个文件最终将更新到最新版本，成为当时的公共修订版本。svn commit不做任何事情，svn update将会取得最新的版本到工作拷贝。
4. 本地已修改且不是最新的, 这个文件在工作目录和版本库都得到修改。一个svn commit将会失败，这个文件必须首先更新，svn update命令会合并公共和本地修改，如果Subversion不可以自动完成，将会让用户解决冲突。

### 5.2 修订版本关键字

这些关键字可以用来代替 `--revision(r)` 的数字参数，这会被 Subversion 解释到特定的修订版本号。

+ `HEAD` : 版本库中最新的（或者是 "最年轻的）版本。
+ `BASE` : 工作拷贝中一个条目的修订版本号，如果这个版本在本地修改了，则“BASE版本”就是这个条目在本地未修改的版本。
+ `COMMITTED` : 项目最近修改的修订版本，与BASE相同或更早。
+ `PREV` : 一个项目最后修改版本之前的那个版本，技术上可以认为是COMMITTED -1。

### 5.3 SVN 和 Git

我感觉 SVN 和 Git 设计的本质区别在于 SVN 核心点在于它的目录，而 Git 的核心点在于它的 Commit(结点)。这也就说明了 SVN 可以 checkout 某一个目录，Git 不行。SVN 的分支可以理解成一个目录，而 Git 的分支只不过是某个 Commit 上的快照而已。

实时更新在: http://linuxwiki.github.io/ProductivityTools/svn.html
