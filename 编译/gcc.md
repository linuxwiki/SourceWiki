## 一、参数选项

### 1.1 编译，链接

* -E filename.c: 预处理，不生成文件。可以查看一段代码预处理后的代码。
* -C: 在预处理的时候,不删除注释信息,一般和 **-E: 使用，用来分析代码。
* -S filename.c: 预处理、编译，生成汇编代码。
* -c filename.c: 预处理、编译、汇编，生成目标文件。
* -M: 生成文件关联的信息(也就是目标文件依赖的源代码)。
* -MM: 和上面的那个一样，但是它将忽略由 #include<file> 造成的依赖关系。
* -llibrary: 指定编译所需的库。
* -Ldir: 指定编译搜索库的路径。
* -g: 在编译的时候产生调试信息。
* -static: 禁止使用动态库。
* -share: 尽量使用动态库。
* -w: 不生成任何警告信息。
* -Wall: 生成所有警告信息。
* -pedantic-error: 允许发出ANSI C标准所列的全部错误信息
* -werror: 把所有的告警信息转化为错误信息，并在告警发生时终止编译过程。
* -o: 生成目标文件。eg: gcc -o hello.asm -S hello.c
* -ansi: 关闭 gnu c 中与 ansi c 不兼容的特性, 激活 ansic的 专有特性(包括禁止一些 asm inline typeof 关键字, 以及 UNIX,vax 等预处理宏)。
* fno-asm: 此选项实现 ansi 选项的功能的一部分，它禁止将 asm, inline 和 typeof 用作关键字。
* -undef: 取消对任何非标准宏的定义。
* -Idir: 指定头文件查找目录(如果没找到，再去缺省的目录中找)。
* -nostdinc: 规定不在 g++ 指定的标准路经中搜索, 但仍在其他路径中搜索。
* -nostdin C++: 规定不在 g++ 指定的标准路经中搜索, 但仍在其他路径中搜索。
* -Wno-invalid-offsetof: -Wno-invalid-offsetof (C++ and Objective-C++ only), Suppress warnings from applying the ‘offsetof’ macro to a non-POD type.

### 1.2 优化选项

* -O0: 关闭所有优选项，不做程序优化，默认等级。
* -O1: 这是最基本的优化等级。编译器会在不花费太多编译时间的同时试图生成更快更小的代码。这些优化是非常基础的，但一般这些任务肯定能顺利完成。
* -O2: O1的进阶。这是推荐的优化等级，除非你有特殊的需求。-O2会比-O1启用多一些标记。设置了-O2后，编译器会试图提高代码性能而不会增大体积和大量占用的编译时间。
* -O3: 是最高最危险的优化等级。用这个选项会延长编译代码的时间，并且在使用gcc4.x的系统里不应全局启用。自从3.x版本以来gcc的行为已经有了极大地改变。在3.x，-O3生成的代码也只是比-O2快一点点而已，而gcc4.x中还未必更快。用-O3来编译所有的软件包将产生更大体积更耗内存的二进制文件，大大增加编译失败的机会或不可预知的程序行为（包括错误）。这样做将得不偿失，记住过犹不及。在gcc 4.x.中使用-O3是不推荐的。

## 二、参考资料

* [GCC 编译优化指南](http://www.jinbuguo.com/linux/optimize_guide.html)
* [GCC Command-Line Options](http://tigcc.ticalc.org/doc/comopts.html)
* [GCC参数详解](http://www.cppblog.com/seman/archive/2005/11/30/1440.html)
* [GCC编译选项分析](http://www.cnblogs.com/showna/articles/1013401.html)
