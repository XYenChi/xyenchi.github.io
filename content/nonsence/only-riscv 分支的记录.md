---
title: "only-riscv 分支的记录"   
date: "2024-04-22T11:30:03+00:00"   
draft: false
---
目前成功构建的 patch ：`RISC-V: Fix VL operand bug in VSETVL PASS[PR110264]`   
此patch`RISC-V: Bugfix for RVV integer reduction in ZVE32/64.`构建失败，但是`RISC-V: Fix one typo for reduc expand GET_MODE_CLASS`可以修复。      
中间夹杂了 `RISC-V:Add float16 tuple type support` ,此构建失败报：   
```
../.././gcc/gcc/rtl.h:316:31: warning: ‘rtx_def::mode’ is too small to hold all values of ‘enum machine_mode’
  316 |   ENUM_BITFIELD(machine_mode) mode : 8;
      |                               ^~~~
In file included from ../.././gcc/gcc/genemit.cc:25:
../.././gcc/gcc/rtl.h:316:31: warning: ‘rtx_def::mode’ is too small to hold all values of ‘enum machine_mode’
  316 |   ENUM_BITFIELD(machine_mode) mode : 8;
      |                               ^~~~
In file included from ../.././gcc/gcc/genextract.cc:22:
../.././gcc/gcc/rtl.h: In member function ‘long unsigned int subreg_shape::unique_id() const’:
../.././gcc/gcc/rtl.h:2160:37: error: static assertion failed: MAX_MACHINE_MODE <= 256
 2160 |   { STATIC_ASSERT (MAX_MACHINE_MODE <= 256); }
../.././gcc/gcc/system.h:850:19: note: in definition of macro ‘STATIC_ASSERT’
  850 |   static_assert ((X), #X)
      |                   ^
../.././gcc/gcc/rtl.h:2160:37: note: the comparison reduces to ‘(284 <= 256)’
 2160 |   { STATIC_ASSERT (MAX_MACHINE_MODE <= 256); }
../.././gcc/gcc/system.h:850:19: note: in definition of macro ‘STATIC_ASSERT’
  850 |   static_assert ((X), #X)
      |                   ^
In file included from ../.././gcc/gcc/genemit.cc:22:
../.././gcc/gcc/rtl.h: In member function ‘long unsigned int subreg_shape::unique_id() const’:
../.././gcc/gcc/rtl.h:2160:37: error: static assertion failed: MAX_MACHINE_MODE <= 256
 2160 |   { STATIC_ASSERT (MAX_MACHINE_MODE <= 256); }
../.././gcc/gcc/system.h:850:19: note: in definition of macro ‘STATIC_ASSERT’
  850 |   static_assert ((X), #X)
      |                   ^
../.././gcc/gcc/rtl.h:2160:37: note: the comparison reduces to ‘(284 <= 256)’
 2160 |   { STATIC_ASSERT (MAX_MACHINE_MODE <= 256); }
../.././gcc/gcc/system.h:850:19: note: in definition of macro ‘STATIC_ASSERT’
  850 |   static_assert ((X), #X)
      |                   ^
In file included from ../.././gcc/gcc/genpeep.cc:25:
../.././gcc/gcc/rtl.h:316:31: warning: ‘rtx_def::mode’ is too small to hold all values of ‘enum machine_mode’
  316 |   ENUM_BITFIELD(machine_mode) mode : 8;
      |                               ^~~~
In file included from ../.././gcc/gcc/rtl.cc:31:
../.././gcc/gcc/rtl.h:316:31: warning: ‘rtx_def::mode’ is too small to hold all values of ‘enum machine_mode’
  316 |   ENUM_BITFIELD(machine_mode) mode : 8;
      |                               ^~~~
In file included from ../.././gcc/gcc/rtl.cc:28:
../.././gcc/gcc/rtl.h: In member function ‘long unsigned int subreg_shape::unique_id() const’:
../.././gcc/gcc/rtl.h:2160:37: error: static assertion failed: MAX_MACHINE_MODE <= 256
 2160 |   { STATIC_ASSERT (MAX_MACHINE_MODE <= 256); }
../.././gcc/gcc/system.h:850:19: note: in definition of macro ‘STATIC_ASSERT’
  850 |   static_assert ((X), #X)
      |                   ^
../.././gcc/gcc/rtl.h:2160:37: note: the comparison reduces to ‘(284 <= 256)’
 2160 |   { STATIC_ASSERT (MAX_MACHINE_MODE <= 256); }
../.././gcc/gcc/system.h:850:19: note: in definition of macro ‘STATIC_ASSERT’
  850 |   static_assert ((X), #X)
      |                   ^
In file included from ../.././gcc/gcc/genpeep.cc:22:
../.././gcc/gcc/rtl.h: In member function ‘long unsigned int subreg_shape::unique_id() const’:
../.././gcc/gcc/rtl.h:2160:37: error: static assertion failed: MAX_MACHINE_MODE <= 256
 2160 |   { STATIC_ASSERT (MAX_MACHINE_MODE <= 256); }
../.././gcc/gcc/system.h:850:19: note: in definition of macro ‘STATIC_ASSERT’
  850 |   static_assert ((X), #X)
      |                   ^
../.././gcc/gcc/rtl.h:2160:37: note: the comparison reduces to ‘(284 <= 256)’
 2160 |   { STATIC_ASSERT (MAX_MACHINE_MODE <= 256); }
../.././gcc/gcc/system.h:850:19: note: in definition of macro ‘STATIC_ASSERT’
  850 |   static_assert ((X), #X)
      |                   ^
In file included from ../.././gcc/gcc/genautomata.cc:111:
../.././gcc/gcc/rtl.h:316:31: warning: ‘rtx_def::mode’ is too small to hold all values of ‘enum machine_mode’
  316 |   ENUM_BITFIELD(machine_mode) mode : 8;
      |                               ^~~~
In file included from ../.././gcc/gcc/genautomata.cc:108:
../.././gcc/gcc/rtl.h: In member function ‘long unsigned int subreg_shape::unique_id() const’:
../.././gcc/gcc/rtl.h:2160:37: error: static assertion failed: MAX_MACHINE_MODE <= 256
 2160 |   { STATIC_ASSERT (MAX_MACHINE_MODE <= 256); }
../.././gcc/gcc/system.h:850:19: note: in definition of macro ‘STATIC_ASSERT’
  850 |   static_assert ((X), #X)
      |                   ^
../.././gcc/gcc/rtl.h:2160:37: note: the comparison reduces to ‘(284 <= 256)’
 2160 |   { STATIC_ASSERT (MAX_MACHINE_MODE <= 256); }
../.././gcc/gcc/system.h:850:19: note: in definition of macro ‘STATIC_ASSERT’
  850 |   static_assert ((X), #X)
      |                   ^
g++ -c   -g -O2   -DIN_GCC  -DCROSS_DIRECTORY_STRUCTURE   -fno-exceptions -fno-rtti -fasynchronous-unwind-tables -W -Wall -Wno-narrowing -Wwrite-strings -Wcast-qual -Wmissing-format-attribute -Woverloaded-virtual -pedantic -Wno-long-long -Wno-variadic-macros -Wno-overlength-strings   -DHAVE_CONFIG_H  -DGENERATOR_FILE -I. -Ibuild -I../.././gcc/gcc -I../.././gcc/gcc/build -I../.././gcc/gcc/../include  -I../.././gcc/gcc/../libcpp/include  \
        -o build/gensupport.o ../.././gcc/gcc/gensupport.cc
In file included from ../.././gcc/gcc/genpreds.cc:27:
../.././gcc/gcc/rtl.h:316:31: warning: ‘rtx_def::mode’ is too small to hold all values of ‘enum machine_mode’
  316 |   ENUM_BITFIELD(machine_mode) mode : 8;
      |                               ^~~~
In file included from ../.././gcc/gcc/genoutput.cc:90:
../.././gcc/gcc/rtl.h:316:31: warning: ‘rtx_def::mode’ is too small to hold all values of ‘enum machine_mode’
  316 |   ENUM_BITFIELD(machine_mode) mode : 8;
      |                               ^~~~
In file included from ../.././gcc/gcc/genpreds.cc:24:
../.././gcc/gcc/rtl.h: In member function ‘long unsigned int subreg_shape::unique_id() const’:
../.././gcc/gcc/rtl.h:2160:37: error: static assertion failed: MAX_MACHINE_MODE <= 256
 2160 |   { STATIC_ASSERT (MAX_MACHINE_MODE <= 256); }
../.././gcc/gcc/system.h:850:19: note: in definition of macro ‘STATIC_ASSERT’
  850 |   static_assert ((X), #X)
      |                   ^
../.././gcc/gcc/rtl.h:2160:37: note: the comparison reduces to ‘(284 <= 256)’
 2160 |   { STATIC_ASSERT (MAX_MACHINE_MODE <= 256); }
../.././gcc/gcc/system.h:850:19: note: in definition of macro ‘STATIC_ASSERT’
  850 |   static_assert ((X), #X)
      |                   ^
In file included from ../.././gcc/gcc/genopinit.cc:25:
../.././gcc/gcc/rtl.h:316:31: warning: ‘rtx_def::mode’ is too small to hold all values of ‘enum machine_mode’
  316 |   ENUM_BITFIELD(machine_mode) mode : 8;
      |                               ^~~~
In file included from ../.././gcc/gcc/genoutput.cc:87:
../.././gcc/gcc/rtl.h: In member function ‘long unsigned int subreg_shape::unique_id() const’:
../.././gcc/gcc/rtl.h:2160:37: error: static assertion failed: MAX_MACHINE_MODE <= 256
 2160 |   { STATIC_ASSERT (MAX_MACHINE_MODE <= 256); }
../.././gcc/gcc/system.h:850:19: note: in definition of macro ‘STATIC_ASSERT’
  850 |   static_assert ((X), #X)
      |                   ^
../.././gcc/gcc/rtl.h:2160:37: note: the comparison reduces to ‘(284 <= 256)’
 2160 |   { STATIC_ASSERT (MAX_MACHINE_MODE <= 256); }
../.././gcc/gcc/system.h:850:19: note: in definition of macro ‘STATIC_ASSERT’
  850 |   static_assert ((X), #X)
      |                   ^
In file included from ../.././gcc/gcc/genopinit.cc:22:
../.././gcc/gcc/rtl.h: In member function ‘long unsigned int subreg_shape::unique_id() const’:
../.././gcc/gcc/rtl.h:2160:37: error: static assertion failed: MAX_MACHINE_MODE <= 256
 2160 |   { STATIC_ASSERT (MAX_MACHINE_MODE <= 256); }
../.././gcc/gcc/system.h:850:19: note: in definition of macro ‘STATIC_ASSERT’
  850 |   static_assert ((X), #X)
      |                   ^
../.././gcc/gcc/rtl.h:2160:37: note: the comparison reduces to ‘(284 <= 256)’
 2160 |   { STATIC_ASSERT (MAX_MACHINE_MODE <= 256); }
../.././gcc/gcc/system.h:850:19: note: in definition of macro ‘STATIC_ASSERT’
  850 |   static_assert ((X), #X)
      |                   ^
In file included from ../.././gcc/gcc/genrecog.cc:112:
../.././gcc/gcc/rtl.h:316:31: warning: ‘rtx_def::mode’ is too small to hold all values of ‘enum machine_mode’
  316 |   ENUM_BITFIELD(machine_mode) mode : 8;
      |                               ^~~~
make[2]: *** [Makefile:2825: build/genextract.o] Error 1
make[2]: *** Waiting for unfinished jobs....
In file included from ../.././gcc/gcc/genrecog.cc:109:
../.././gcc/gcc/rtl.h: In member function ‘long unsigned int subreg_shape::unique_id() const’:
../.././gcc/gcc/rtl.h:2160:37: error: static assertion failed: MAX_MACHINE_MODE <= 256
 2160 |   { STATIC_ASSERT (MAX_MACHINE_MODE <= 256); }
../.././gcc/gcc/system.h:850:19: note: in definition of macro ‘STATIC_ASSERT’
  850 |   static_assert ((X), #X)
      |                   ^
../.././gcc/gcc/rtl.h:2160:37: note: the comparison reduces to ‘(284 <= 256)’
 2160 |   { STATIC_ASSERT (MAX_MACHINE_MODE <= 256); }
../.././gcc/gcc/system.h:850:19: note: in definition of macro ‘STATIC_ASSERT’
  850 |   static_assert ((X), #X)
      |                   ^
In file included from ../.././gcc/gcc/genattrtab.cc:109:
../.././gcc/gcc/rtl.h:316:31: warning: ‘rtx_def::mode’ is too small to hold all values of ‘enum machine_mode’
  316 |   ENUM_BITFIELD(machine_mode) mode : 8;
      |                               ^~~~
make[2]: *** [Makefile:2825: build/genpeep.o] Error 1
make[2]: *** [Makefile:2825: build/genemit.o] Error 1
In file included from ../.././gcc/gcc/genattrtab.cc:106:
../.././gcc/gcc/rtl.h: In member function ‘long unsigned int subreg_shape::unique_id() const’:
../.././gcc/gcc/rtl.h:2160:37: error: static assertion failed: MAX_MACHINE_MODE <= 256
 2160 |   { STATIC_ASSERT (MAX_MACHINE_MODE <= 256); }
../.././gcc/gcc/system.h:850:19: note: in definition of macro ‘STATIC_ASSERT’
  850 |   static_assert ((X), #X)
      |                   ^
../.././gcc/gcc/rtl.h:2160:37: note: the comparison reduces to ‘(284 <= 256)’
 2160 |   { STATIC_ASSERT (MAX_MACHINE_MODE <= 256); }
../.././gcc/gcc/system.h:850:19: note: in definition of macro ‘STATIC_ASSERT’
  850 |   static_assert ((X), #X)
      |                   ^
In file included from ../.././gcc/gcc/read-rtl.cc:34:
../.././gcc/gcc/rtl.h:316:31: warning: ‘rtx_def::mode’ is too small to hold all values of ‘enum machine_mode’
  316 |   ENUM_BITFIELD(machine_mode) mode : 8;
      |                               ^~~~
In file included from ../.././gcc/gcc/read-rtl.cc:31:
../.././gcc/gcc/rtl.h: In member function ‘long unsigned int subreg_shape::unique_id() const’:
../.././gcc/gcc/rtl.h:2160:37: error: static assertion failed: MAX_MACHINE_MODE <= 256
 2160 |   { STATIC_ASSERT (MAX_MACHINE_MODE <= 256); }
../.././gcc/gcc/system.h:850:19: note: in definition of macro ‘STATIC_ASSERT’
  850 |   static_assert ((X), #X)
      |                   ^
../.././gcc/gcc/rtl.h:2160:37: note: the comparison reduces to ‘(284 <= 256)’
 2160 |   { STATIC_ASSERT (MAX_MACHINE_MODE <= 256); }
../.././gcc/gcc/system.h:850:19: note: in definition of macro ‘STATIC_ASSERT’
  850 |   static_assert ((X), #X)
      |                   ^
make[2]: *** [Makefile:2825: build/rtl.o] Error 1
make[2]: *** [Makefile:2825: build/genopinit.o] Error 1
make[2]: *** [Makefile:2825: build/genoutput.o] Error 1
make[2]: *** [Makefile:2825: build/genpreds.o] Error 1
make[2]: *** [Makefile:2825: build/read-rtl.o] Error 1
make[2]: *** [Makefile:2825: build/genrecog.o] Error 1
make[2]: *** [Makefile:2825: build/genattrtab.o] Error 1
make[2]: *** [Makefile:2825: build/genautomata.o] Error 1
In file included from ../.././gcc/gcc/gensupport.cc:24:
../.././gcc/gcc/rtl.h:316:31: warning: ‘rtx_def::mode’ is too small to hold all values of ‘enum machine_mode’
  316 |   ENUM_BITFIELD(machine_mode) mode : 8;
      |                               ^~~~
In file included from ../.././gcc/gcc/gensupport.cc:21:
../.././gcc/gcc/rtl.h: In member function ‘long unsigned int subreg_shape::unique_id() const’:
../.././gcc/gcc/rtl.h:2160:37: error: static assertion failed: MAX_MACHINE_MODE <= 256
 2160 |   { STATIC_ASSERT (MAX_MACHINE_MODE <= 256); }
../.././gcc/gcc/system.h:850:19: note: in definition of macro ‘STATIC_ASSERT’
  850 |   static_assert ((X), #X)
      |                   ^
../.././gcc/gcc/rtl.h:2160:37: note: the comparison reduces to ‘(284 <= 256)’
 2160 |   { STATIC_ASSERT (MAX_MACHINE_MODE <= 256); }
../.././gcc/gcc/system.h:850:19: note: in definition of macro ‘STATIC_ASSERT’
  850 |   static_assert ((X), #X)
      |                   ^
make[2]: *** [Makefile:2825: build/gensupport.o] Error 1
```    
可能是这俩的问题吧：    
`Bump up precision size to 16 bits.`   
`Machine_Mode: Extend machine_mode from 8 to 16 bits`   

