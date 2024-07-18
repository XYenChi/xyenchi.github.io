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
### 简单总结一下热心大佬们介绍的知识
