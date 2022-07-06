---
title: "Port gprofng for RISC-V"
draft: false
---
### 简单介绍一下 gprofng 的功能   
由于 gprofng 可以对可执行程序直接进行分析不需要重新编译源码，所以对闭源软件进行性能评测也很方便。    
1. 对 C、C++、Java 或者 Scala 程序进行性能分析    
2. 支持 Pthread、OpenMP 和 Java thread 多线程编程模型   
3. gprofng 主要数据收集命令 gprofng collect app 运行时一般使用程序计数器（PC）采样 (Sample)  
### 简单介绍一下 gprofng 的安装方法   
1. 直接用各大发行版的软件包管理器安装
2. 从源码开始编译安装    
        https://github.com/bminor/binutils-gdb/tree/master/gprofng
### 简单介绍一下 gprofng 的使用方法    
首先收集数据：
```shell
gprofng collect app {your app}
```   
eg.   
```shell
gprofng collect app echo 1
```   
上述命令会生成 test.n.er 文件夹，里面是收集的数据。    
然后进行处理：   
```shell
gprofng display text -func -exp -head test.{times}.er/
```   
### 简单介绍一下 port 过程干了啥   
#### 添加 binutils-gdb 的 --enable-gprofng 支持的架构   
[configure.ac](https://sourceware.org/git/?p=binutils-gdb.git;a=blob;f=configure.ac;h=a390639bfa90a8013da0b6c65aa6f848f1b26759;hb=91b999864f9831287f2ea6f960e8fa98e7b13ee9)中判断是否开启构建 gprofng 的 target 里面添加 riscv64 的 [^1]triplet。   

[gnu configure.ac 的介绍官网链接](https://www.gnu.org/software/autoconf/manual/autoconf-2.60/html_node/Writing-configure_002eac.html)   
虽然很短一段但是翻译过来放在我的博客文章中却可以显得我的博客很长：   
新建名为 *configure.ac* 、包含调用 Autoconf 用来检测软件包所需或者可以使用的系统特性的宏的文件来给软件包生成 *configure* 脚本。   
从描述上看，Autoconf 已有的宏可以检测多种特性；详见[已有检测](https://www.gnu.org/software/autoconf/manual/autoconf-2.60/html_node/Existing-Tests.html#Existing-Tests)   
也可以使用 Autoconf 模版来生成一般性检测宏；详见[写检测](https://www.gnu.org/software/autoconf/manual/autoconf-2.60/html_node/Writing-Tests.html#Writing-Tests)   
对于特殊的特性，*configure.ac* 可能需要手写 shell 命令；详见[可移植 shell](https://www.gnu.org/software/autoconf/manual/autoconf-2.60/html_node/Portable-Shell.html#Portable-Shell)   
*autoscan* 程序可以帮助你快速入门写 *configure.ac* ，详见[autoscan 调用](https://www.gnu.org/software/autoconf/manual/autoconf-2.60/html_node/autoscan-Invocation.html#autoscan-Invocation);   
同理在 gprofng 文件夹中的 configure.ac 中也添加 RISC-V 架构相关配置信息。   

#### 添加 CPU 厂商信息 

根据 `linux/arch/riscv/include/asm/vendorid_list.h` ：   

在 `binutils-gdb/gprofng/common/core_pcbe.c` 的 core_pcbe_init 函数里添加厂商信息
   
core_pcbe_cpuref 观察到 arm 返回的是空字符，RISC-V 也没有 model 的区别。
   
在 `binutils-gdb/gprofng/common/cpuid.c` 中使用 hwprobe 。   
RISC-V Optimization Guide 中介绍了 [hwprobe](https://riscv-optimization-guide-riseproject-c94355ae3e6872252baa952524.gitlab.io/riscv-optimization-guide.html#_detecting_risc_v_extensions_on_linux) 的好处。   
邱老师也对 RISC-V Optimization Guide 进行了详细的解读；[bilibili 链接](https://www.bilibili.com/video/BV1Ft421t7PS)   

#### 添加与 RISC-V 架构相关的宏定义   
这部分都是根据其他架构稍加改动添加的。   

#### 添加架构相关的代码   
在 `gprofng/libcollector/unwind.c` 中根据 glibc 和 RISC-V 寄存器的名称编号实现其他架构相关功能。   
[glibc 的 uc_mcontext 声明](https://github.com/bminor/glibc/blob/master/sysdeps/unix/sysv/linux/riscv/sys/ucontext.h)   

#### 添加 cpu frequency 常量   
`gprofng/src/collctrl.cc` 中：
1.加入 ` cpu_info.cpu_clk_freq = 1000;`    
目前没有找到 RISC-V 简单的获取 cpu 频率的方法，所以邱老师建议我先写常量，等下一步完善。   
2.在打开 /proc/cpuinfo 的过程中根据 RISC-V 的 /proc/cpuinfo 的内容加入‘mvendorid’的字符串匹配。   

### 简单总结一下热心大佬们介绍的知识    
南盘江计划的参与小伙伴推荐的 [chip wiki 网站](https://en.wikichip.org/wiki/risc-v/registers)。   
酸鸽介绍的 [glibc uc_mcontext](https://github.com/bminor/glibc/blob/master/sysdeps/unix/sysv/linux/riscv/sys/ucontext.h)   
cyy介绍的 [hwprobe](https://docs.kernel.org/arch/riscv/hwprobe.html)   
感谢邱老师教我移植软件的方法还帮忙修好了glic的调用过程。终于迈出了提交patch的第一步，很难想象没有大家的帮助，我能做的有多少。   
虽然到现在对 gprofng 整个工作的内部细节了解得没有很透彻，但是先记录下来现在干的事情等以后变强了继续完善。   

[^1]: ABI 中规定 triplet 格式为：cpu-vendor-os   