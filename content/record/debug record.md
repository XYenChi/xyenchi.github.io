---
title: "debug record"
date: "2023-11-06T12:59:38+08:00"
url: "/debug_record/"
draft: false
---
### 工具链
[riscv-gnu-toolchain](https://github.com/riscv-collab/riscv-gnu-toolchain)
### 源码   
https://github.com/gcc-mirror/gcc/blob/master/gcc/common/config/riscv/riscv-common.cc   
### 方法
`git clone https://github.com/riscv-collab/riscv-gnu-toolchain.git`   
`./configure --prefix="$PWD/opt" --with-arch=rv64gc --with-abi=lp64d`   
`make -j $(nproc)`   
`make linux -j $(nproc)`   
`gdb opt/riscv64-unknown-linux-gnu-gcc`   

没想到想办法实现了好几天的 `-march=rv64gcv0p7` 解析成 `rv64i2p1_m2p0_a2p1_f2p2 _d2p2 _c2p0 _v0p7` 而不是 `rv64i2p1_m2p0_a2p1_f2p2_d2p2_c2p0_v0p7_zicsr2p0_zifencei2p0_zve32f1p0_zve32x1p0_zve64d1p0_zve64f1p0_zve64x1p0_zvl128b1p0_zvl32b1p0_zvl64b1p0` 因为 T head 实现方法是 `-march=rv64gcXthreadVector` 而搁置了。   

初期是在 `handle_implied_ext` 前面判断版本。但是发现参数 `*ext` 在与 `riscv_implied_info[]` 比较，没有获取大小版本的机会，于是引入了一个参数指给 `riscv_ext_version_table[]`。还用过 `get_default_version`，始终卡在不知道获取了哪个扩展的 major_version = 2, minor_version = 0。   

继而想在`parse`调用 `handle_implied_ext` 之前判断出大小版本返回 `implied_version_p = false`。但是使用前文中的 `itr` 调用 `lookup()`始终不对。自此编程苦手烂尾。