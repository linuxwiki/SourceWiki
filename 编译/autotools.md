**本文档整理自: [Autotools Tutorial: Part II](https://www.lrde.epita.fr/~adl/dl/autotools.pdf)。**

## 一、Hello World

### 1.1 创建文件

*src/main.c*

    #include <config.h>
    #include <stdio.h>
    
    int main(void)
    {
        puts ("Hello World!");
        puts ("This is " PACKAGE_STRING ".");
        return 0;
    }


*configure.ac*

    AC_INIT([amhello], [1.0], [bug-report@address])
    AM_INIT_AUTOMAKE([foreign -Wall -Werror])
    AC_PROG_CC
    AC_CONFIG_HEADER([config.h])
    AC_CONFIG_FILES([Makefile src/Makefile])
    AC_OUTPUT

*Makefile.am*

    SUBDIRS = src

*src/Makefile.am*

    bin_PROGRAMS = hello
    hello_SOURCES = main.c

此时的文件结构为(`ls -R`):

    .:
    Makefile.am  configure.ac  src
    
    src:
    Makefile.am  main.c


### 1.2 autoreconf --install

    $: autoreconf --install
    configure.ac:2: installing `./install-sh'
    configure.ac:2: installing `./missing'
    src/Makefile.am: installing `./depcomp'

此时的文件结构为(`ls -R`):

    .:
    Makefile.am  aclocal.m4      config.h.in  configure.ac  install-sh  src
    Makefile.in  autom4te.cache  configure    depcomp       missing
    
    autom4te.cache:
    output.0  output.1  requests  traces.0  traces.1
    
    src:
    Makefile.am  Makefile.in  main.c

+ Makefile.in config.h.in configure* src/Makefile.in : expected configuration templates.
+ aclocal.m4 : denitions for third-party macros used in *configure.ac*.
+ depcomp* install-sh* missing* : auxiliary tools used during the build.
+ autom4te.cache/ autom4te.cache/* : Autotools cache files

### 1.3 `./configure`

    checking for a BSD-compatible install... /usr/bin/install -c
    checking whether build environment is sane... yes
    checking for a thread-safe mkdir -p... /bin/mkdir -p
    checking for gawk... gawk
    checking whether make sets $(MAKE)... yes
    checking for gcc... gcc
    ...
    configure: creating ./config.status
    config.status: creating Makefile
    config.status: creating src/Makefile
    config.status: creating config.h
    config.status: executing depfiles commands

### 1.4 `make`

    /usr/bin/make  all-recursive
    make[1]: Entering directory `/home/zhangjie/TempFile/src'
    Making all in src
    make[2]: Entering directory `/home/zhangjie/TempFile/src/src'
    gcc -DHAVE_CONFIG_H -I. -I..     -g -O2 -MT main.o -MD -MP -MF .deps/main.Tpo -c -o main.o main.c
    mv -f .deps/main.Tpo .deps/main.Po
    gcc  -g -O2   -o hello main.o  
    make[2]: Leaving directory `/home/zhangjie/TempFile/src/src'
    make[2]: Entering directory `/home/zhangjie/TempFile/src'
    make[2]: Leaving directory `/home/zhangjie/TempFile/src'
    make[1]: Leaving directory `/home/zhangjie/TempFile/src'

### 1.5 编译、链接成功，运行程序: `src/hello`

    $: src/hello 
    Hello World!
    This is amhello 1.0.

### 1.6 源码打包: `make distcheck`

    $: make distcheck
    ...
    =============================================
    amhello-1.0 archives ready for distribution: 
    amhello-1.0.tar.gz
    =============================================

### 1.7 解包，查看文件内容

    $ tar ztf amhello-1.0.tar.gz 
    amhello-1.0/
    amhello-1.0/Makefile.am
    amhello-1.0/aclocal.m4
    amhello-1.0/configure.ac
    amhello-1.0/src/
    amhello-1.0/src/Makefile.am
    amhello-1.0/src/main.c
    amhello-1.0/src/Makefile.in
    amhello-1.0/config.h.in
    amhello-1.0/Makefile.in
    amhello-1.0/missing
    amhello-1.0/configure
    amhello-1.0/depcomp
    amhello-1.0/install-sh

## 二、Autotools 核心介绍

### 2.1 GNU AutoConf

+ `autoconf` : Create *configure* from *configure.ac* .
+ `autoheader` : Create *config.h.in* from *configure.ac* .
+ `autoreconf` : Run all tools in the right order.
+ `autoscan` : Scan sources for common portability problems, and related macros missing from *configure.ac* .
+ `autoupdate` : Update obsolete macros in *configure.ac* .
+ `ifnames` : Gather identiers from all `#if/#ifdef/...` directives.
+ `autom4te` :  The heart of Autoconf. It drives M4 and implements the
features used by most of the above tools. Useful for
creating more than just *configure* files.

### 2.2 GNU AutoMake

+ `automake` : Create *Makefile.in*s from *Makefile.am*s and *configure.ac* .
+ `aclocal` : Scan `configure.ac` for uses of third-party macros, and
gather denitions in `aclocal.m4`.

### 2.3 autoreconf

![autoreconf](../imaages/autoreconf.jpg)

在实际应用中:

+ 没必要记住所有工具之间的关系；
+ 在初始化的时候，使用 `autoreconf --install`；
+ 修改输出(配置)文件的时候，重建规则；
+ 你需对解每个工具产生的错误都有一个粗糙的理解（至少要知道是谁产生的错误）。

## 三、Hello World 讲解

### 3.1 configure.ac

    AC_INIT([amhello], [1.0], [bug-report@address])
    AM_INIT_AUTOMAKE([foreign -Wall -Werror])
    AC_PROG_CC
    AC_CONFIG_HEADER([config.h])
    AC_CONFIG_FILES([Makefile src/Makefile])
    AC_OUTPUT

+ `AC_INIT`: Initialize *Autoconf*. Specify package's name, version number, and bug-report address.
+ `AM_INIT_AUTOMAKE`: Initialize *Automake*. Turn on all Automake warnning and report them as errors. This is a foreign package.
+ `AC_PROG_CC`: Check for a C compiler.
+ `AC_CONFIG_HEADER([config.h])`: Declare *config.h* as output header.
+ `AC_CONFIG_FILES`: Declare *Makefile* and *src/Makefile* as output files.
+ `AC_OUTPUT`: Actually output all declared files.

### 3.2 Makefile.am

    SUBDIRS = src

+ Build recursively in *src/* .
+ Nothing else is declared for the current directory.(The top-level *Makefile.am* is usually short.)

### 3.2 src/Makefile.am

    bin_PROGRAMS = hello
    hello_SOURCES = main.c

+ We are building some programs.
+ These programs will be installed in *bindir* (注意看下面tip的结构).
+ There is only one program to build: *hello*.
+ To create *hello*, just compile *main.c*.

**tip**: Standard File System Hierarchy:

![](/attach/compile/sys_hierarchy.jpg)

## 四、使用 Autoconf

### 4.1 从 configure.ac 到 configure 和 config.h.in

+ "autoconf is a macro processor.
+ It converts *configure.ac*, which is a shell script using macro instructions, into *congfiure*, a full-fledged shell script.
+ Autoconf offers many macros to perform common conguration checks.
+ It is not uncommon to have a *configure.ac* without shell construct, using only macros.
+ While processing *configure.ac* it is also possible to trace the occurrences of macros. This is how "autoheader" creates *config.h.in* . It just looks for the macros that #define symbols.
+ The real macro processor actually is GNU M4. Autoconf offers some infrastructure on top of that, plus the pool of macros.

### 4.2 探索 M4

*exmple.m4:*

    m4_define(NAME1, Harry)
    m4_define(NAME2, Sally)
    m4_define(MET, $1 met $2)
    MET(NAME1, NAME2)

output:

    $: m4 -P example.m4 
    # 空白
    # 空白
    # 空白
    Harry met Sally

M4 是一个宏处理器，将输出拷贝到输出。可以内嵌，也可以用户自定义，除了宏展开以外，M4还有一些内建的函数，用来引用文件、执行Unix命令、文本操作、循环等。

这里不详细展开表述 M4，但是要记住 M4 是 Autoconf 的核心，Autoconf 只是封装了一些指令而已。

### 4.3 M4 基础上的 Autoconf

+ 在许多机器上， Auto = M4，和一些预定义的宏。
+ 引用是 `[` 和 `]` (而不是 `'` 和 `'`)。
+ 也因此在 Shell 中做比较使用 `test` ，而不是 `[` 。

        if test "$x" = "$y"; then ...

+ 使用 `AC_DEFUN` 定义宏。

        AC_DEFUN([NAME1], [Harry, Jr.])
        AC_DEFUN([NAME2], [Sally])
        AC_DEFUN([MET], [$1 met $2])
        MET([NAME1], [NAME2])


### 4.3 一个 configure.ac 的结构

    # Prelude
    AC_INIT([PACKAGE], [VERSION], [BUG-REPORT-ADDRESS])
    AM_INIT_AUTOMAKE([foreign -Wall -Werror])
    AC_PREREQ(VERSION) #  Eg. AC_PREREQ([2.65])
    AC_CONFIG_SRCDIR(FILE) # Eg. AC_CONFIG_SRCDIR([src/main.c])
    AC_CONFIG_AUX_DIR(DIRECTORY) # mising, install-sh 等临时文件的目录 eg. AC_CONFIG_AUX_DIR([build-aux]) 
    
    # Checks for programs.
    AC_PROG_CC
    AC_PROG_CXX
    AC_PROG_AWK
    AC_CHECK_PROGS(VAR, PROGS, [VAL-IF-NOT-FOUND]) # Dene VAR to the rst PROGS found, or to VAL-IF-NOT-FOUND otherwise.
    
    # Check for libraries.
    AC_CHECK_LIB(LIBRARY, FUNCT, [ACT-IF-FOUND], [ACT-IF-NOT]) # Check whether LIBRARY exists and contains FUNCT. AC CHECK HEADERS(HEADERS...)Execute ACT-IF-FOUND if it does, ACT-IF-NOT otherwise
    
    # Check for header files.
    AC_CHECK_HEADERS(HEADERS...) # Check for HEADERS and #define HAVE_HEADER_H for each header found. eg. AC_CHECK_HEADERS([sys/param.h unistd.h])
    
    # Check for typedefs, structures, and compiler characteristics.
    
    # Check for library functions.
    
    # Output files.
    AC_CONFIG_HEADERS([config.h])
    AC_CONFIG_FILES([Makefile src/Makefile ...])
    AC_OUTPUT


## 五、使用 Automake

### 5.1 Automake 规则

+ Automake helps creating **portable** and **GNU-standard compliant* *Makefile*s.
    + You may be used to other kinds of build systems.(eg. no VPATH builds, but all objects go into *obj/*).
    + Do not use Automake if you do not like the GNU Build System: Automake will get in your way if you don't fit the mold.
+ "automake" creates **complex** *Makefile.in*s from simple *Makefile.am*s.
    + Consider *Makefile.in*s as internal details.
+ *Makele.am*s follow roughly the same syntax as *Makefile*s however they usually contains only variable denitions.
    + "automake"" creates build rules from these definitions.
    + It's OK to add extra *Makefile* rules in *Makefile.am*: "automake" will preserve them in the output.
    
### 5.2 在 *configure.ac* 中声明 Automake

语法: `AM_INIT_AUTOMAKE([OPTIONS...])])` . eg: `AM_INIT_AUTOMAKE([foreign -Wall -Werror])` .

选项:

+ `-Wall` : 关闭所有警告.
+ `-Werror` : 把警告当成错误.
+ `foreign` : 放宽 GNU 标准要求.
+ `1.11.1` : automake 的最小版本号
+ `dist-bzip2` : Also create tar.bz2 archives during `make dist` and `make distcheck`.
+ `tar-ustar` :  Create tar archives using the ustar format.

`AC_CONFIG_FILES(FILES...)` : Automake 为每一个有 *FILE*.am 的 *FILE* 生成一个 *FILE*.in 文件。

eg: `AC_CONFIG_FILES([Makefile sub/Makefile])` 会生成 *Makefile.am* 和 *sub/Makefile.am*.

### 5.3 声明源文件

    bin_PROGRAMS = foo run-me
    foo_SOURCES = foo.c foo.h print.c print.h
    run_me_SOURCES = run.c run.h print.c

+ These programs will be installed in $(bindir).
+ The sources of each *program* go into *program*_SOURCES.
+ Non-alphanumeric characters are mapped to '_'.
+ Automake automatically computes the list of objects to build and link from these files.
+ Header files are not compiled. We list them only so they get distributed (Automake does not distribute files it does not know about).
+ It's OK to use the same source for two programs.
+ Compiler and linker are inferred from the extensions.

### 5.4 (静态)库

    lib_LIBRARIES = libfoo.a libbar.a
    libfoo_a_SOURCES = foo.c privfoo.h
    libbar_a_SOURCES = bar.c privbar.h
    include_HEADERS = foo.h bar.h

+ These libraries will be installed in $(libdir).
+ Library names must match lib*.a.
+ Public headers will be installed in $(includedir).
+ Private headers are not installed, like ordinary source files.

### 5.5 目录结构

+ 在每个目录下都要有一个 *Makefile* (也就是 *Makefile.am*)。
+ 所有的 *Makefile* 都要在 *configure.ac* 中声明.

        AC_CONFIG_FILES([Makefile lib/Makefile src/Makefile src/dira/Makefile src/dirb/Makefile])

+ 在顶层目录下运行 make.
+ *Makefile.am*s should fix the order in which to recurse directories using the *SUBDIRS* variable.

        file: Makefile.am
        SUBDIRS = lib src

+ The current directory is implicitly built after subdirectories.
+ You can put: .' where you want to override this.

### 5.6 $(srcdir) and VPATH Builds

+ Remember VPATH builds: a source file is not necessary in the current directory.
+ There are two twin trees: the **build tree**, and the **source tree**.
+ *Makefile* and objects files are in the build tree.
+ *Makefile.in*, *Makefile.am*, and source files are in the source tree.
+ If `./configure` is run in the current directory, the two trees are one.
+ In each *Makefile*, 'config.status' will define `$(srcdir)`: the path to the matching source directory.
+ When referring to sources files or targets in Automake variables, you do not have to worry about *source* vs. *build*, because 'make' will check both directories.
+ You may need `$(srcdir)` when specifying flags for tools, or writing custom commands. E.g., to tell the compiler to include headers from *dir/* , you should write `-I$(srcdir)/dir`, not `-Idir`. (`-Idir` would fetch headers from the build tree.)

### 5.7 Convenience Libraries

*lib/Makefile.am*:

    noinst_LIBRARIES = libcompat.a
    libcompat_a_SOURCES = xalloc.c xalloc.h

*src/Makefile.am*:

    bin_PROGRAMS = foo run-me
    foo_SOURCES = foo.c foo.h print.c print.h
    run_me_SOURCES = run.c run.h print.c
    run_me_LDADD = ../lib/libcompat.a
    run_me_CPPFLAGS = -I$(srcdir)/../lib

+ This is a convenience library, used only when building the package.
+ *LDADD* is added when linking all programs.
+ *AM_CPPFLAGS* contains additional preprocessor flags.
+ You can use per-target variables: they apply to a single program

### 5.8 Per-Target Flags

Assuming foo is a program or library:

+ `foo_CFLAGS` : Additional C compiler flags.
+ `foo_CPPFLAGS` : Additional preprocessor flags (`-Is` and `-Ds`).
+ `foo_LDADD` : Additional link objects, `-ls` and `-Ls` (if *foo* is a program).
+ `foo_LIBADD` : Additional link objects, `-ls` and `-Ls` (if *foo* is a library).
+ `foo_LDFLAGS` : Additional linker flags.

The default value for `foo_XXXFLAGS` is `$(AM_XXXFLAGS)`.

Use plain file names to refer to libraries inside your package (keep `-ls` and `-Ls` for external libraries only).

*src/Makefile.am*:

    bin_PROGRAMS = foo run-me
    foo_SOURCES = foo.c foo.h print.c print.h
    run_me_SOURCES = run.c run.h print.c
    run_me_CPPFLAGS = -I$(srcdir)/../lib
    run_me_LDADD = ../lib/libcompat.a $(EFENCELIB)

### 5.9 What Gets Distributed

`make dist` and `make distcheck` create a tarball containing:

+ All sources declared using ... *SOURCES*
+ All headers declared using ... *HEADERS*
+ All scripts declared with *dist ... SCRIPTS*
+ All data les declared with *dist ... DATA*
+ ...
+ Common les such as *ChangeLog*, *NEWS*, etc.
+ See `automake --help` for a list of those files.
+ Extra les or directories listed into *EXTRA_DIST*.

### 5.10 扩展的 Automake 规则

略。

