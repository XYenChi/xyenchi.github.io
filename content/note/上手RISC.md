---
title: "入门开发RISC-V软件的Tips"
draft: true
---

参加几次 WHLUG (Wuhan Linux User Group)观察羊和我进行 RISC-V 相关分享的观众反响进行了新的分享预案。

### 开源
#### 开源软件和自由软件
### Linux
#### 盘点不同的发行版
### git
### RISC-V
### RISC-V 模拟器
#### 构建使用 spike

```shell
git clone https://github.com/riscv-software-src/riscv-isa-sim.git
cd riscv-isa-sim
export RISCV=/opt/riscv (路径自行选择，此处我选的是/opt/riscv)
mkdir build;cd build
../configure –prefix=$RISCV
make -j $(nproc)
sudo make install
```

```shell
git clone https://github.com/riscv-software-src/riscv-pk.git
cd riscv-pk
mkdir build;cd build
../configure --prefix=$RISCV --host=riscv64-unknown-elf
make -j $(nproc)
sudo make install
```
### RISC-V 工具链
#### 构建使用 riscv-gnu-toolchain
```shell
sudo apt-get install autoconf automake autotools-dev curl python3 python3-pip python3-tomli libmpc-dev libmpfr-dev libgmp-dev gawk build-essential bison flex texinfo gperf libtool patchutils bc zlib1g-dev libexpat-dev ninja-build git cmake libglib2.0-dev libslirp-dev

./configure --prefix=$RISCV --with-arch=rv64gc
make -j $(nproc) 2&>1 | tee build.log
```
```shell
./configure --prefix=$RISCV --with-arch=rv64gc
make linux -j $(nproc) 2&>1 | tee build-linux.log
```

```shell
./configure --prefix=$RISCV --enable-llvm --with-arch=rv64gc
make -j $(nproc) 2&>1 | tee build-llvm.log
```

```shell
$RISCV/bin/riscv64-unknown-elf-gcc hello.c -o hello
```

```shell
make build-qemu
$RISCV/bin/qemu-riscv64 hello
```

```shell
$RISCV/bin/spike pk hello
```