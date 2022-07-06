---
title: "观察 sysconf"
date: "2023-12-28T11:30:03+00:00"
draft: false
---
天气勉强晴，空气污染严重。   

今日学习：   
`sysconf()`   
标准C库。行时获取配置信息。   
```
#include <unistd.h>

long sysconf(int name);
```   
-1: 不支持   
0： 相关函数或头文件存在，但会在运行时询问支持程度   
除了0或-1之外的值： 支持   

例子：   
gprofng/src/collctrl.cc
```C
  /* get CPU count and processor clock rate */
  ncpumax = sysconf (_SC_CPUID_MAX);
```   

`clock_getres`   
glibc 2.17前是 Real-time library (librt, -lrt)，之后是标准 C 库 (libc, -lc)。   
```C
#include <time.h>

int clock_getres(clockid_t clockid, struct timespec *_Nullable res);

int clock_gettime(clockid_t clockid, struct timespec *tp);
int clock_settime(clockid_t clockid, const struct timespec *tp);
```
ref：https://man7.org/linux/man-pages/man2/clock_getres.2.html   

在 Michael Kerrisk 写的《Linux/UNIX System Programming Essentials》看到的小技巧：   
不要在写了大量代码之后编译。   
Use a frequent edit-save-build cycle to catch compiler errors early.   
E.g., run the following in a separate window as you edit:   
`$ while inotifywait -q . ; do echo -e '\n\n'; make; done`    
inotifywait is provided in the inotify-tools package   
(The echo command just injects some white space between
each build)