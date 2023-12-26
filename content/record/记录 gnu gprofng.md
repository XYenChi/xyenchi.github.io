---
title: "记录 Gnu Gprofng"
date: 2023-12-01T17:57:55+08:00
draft: true
---

### 构建
`git clone https://github.com/bminor/binutils-gdb.git`   
`cd binutils-gdb`   
`mkdir buildrv && cd buildrv`   
`../configure --enable-gprofng`   
`make`   
### 修改
##### 对 aarch64 进行模仿
###### binutils/gprofng/configure.ac 
添加：    
```shell
    riscv*-*-linux*)
      build_src=true
      build_collector=true
```   
##### binutils/gprofng/common/core_pcbe.c   
因为 RISC-V 没有 cpu model,和 aarch64 一样，所以返回 ""。   

```C
static const char *
core_pcbe_cpuref (void)
{
#if defined(__aarch64__) || defined(__riscv)
  return "";
#elif defined(__i386__) || defined(__x86_64)
  switch (cpuid_getmodel ())
    {
    case 60: /* Haswell */
    case 63:
    case 69:
    case 70:
      return GTXT ("See Chapter 19 of the \"Intel 64 and IA-32 Architectures Software Developer's Manual Volume 3B: System Programming Guide, Part 2\"\nOrder Number: 253669-047US, June 2013");
    case 61: /* Broadwell */
    case 71:
    case 79:
    case 86:
    case 78: /* Skylake */
    case 85:
    case 94:
      return GTXT ("See Chapter 19 of the \"Intel 64 and IA-32 Architectures Software Developer's Manual Volume 3B: System Programming Guide\"");
    default:
      return
      GTXT ("See Chapter 19 of the \"Intel 64 and IA-32 Architectures Software Developer's Manual Volume 3B: System Programming Guide, Part 2\"\nOrder Number: 253669-045US, January 2013");
    }
#else
  return GTXT ("Unknown cpu model");
#endif
}
```   
