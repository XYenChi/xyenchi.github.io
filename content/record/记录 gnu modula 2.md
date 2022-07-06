---
title: "记录 Gnu Modula 2"
date: "2023-11-16T11:30:03+00:00"
draft: false
url: "/gnu/gcc/modula2"
---
Gnu Modula 2 在 gcc 源码目录里，属于处理源代码的前端。根据职业生涯所学从 AST 到 GIMPLE 到 pass 到 RTL, 竟不知从何下手去移植到 RISC-V。   

查询了一下 modula 2 的用法。和 gcc 较为相似，使用 gm2 编译运行 .mod 文件。   

猜测是也能在 riscv-gnu-toolchain 里生成可执行文件，类似于 riscv-gnu-unknown-linux/elf-gcc。     

将 gcc 切换到 devel/modula-2 分支之后，使用 `--with=rv64gc --with-abi-lp64d` 编译了一次，缺少某个库而报错。   
意外的是，如果进入 gcc 文件夹，使用 `--enable-libgm2 GM2_for_target=riscv` 配置一下之后，编译通过了。   
想到修改 makefile 。添加 makefile 的难处在于，gm2 在 gcc 文件夹中，如何使之脱离 gcc 并能够使用。   

