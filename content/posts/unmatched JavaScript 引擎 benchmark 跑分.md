---
title: "Unmatched JavaScript 引擎 Benchmark 跑分"
date: 2022-09-05T12:59:38+08:00
url: "/unmatched/benchmark/JS"
draft: false
---
 unmatched 的系统信息 `Linux milk 5.19.2-arch1-1 #1 SMP PREEMPT Fri, 19 Aug 2022 19:44:19 +0000 riscv64 GNU/Linux` python 版本 :`Python 3.10.6`
###  v8   
arch riscv 的 unmatched 上面并没有 v8 直接装来用，所以需要构建一个。放一个 [v8 官方网站的构建教程](https://v8.dev/docs/build)做参考。但是这样以我的水平并不能糊出来一个 PKGBUILD ，于是在 AUR 库找到了 v8-r 这个包，用`PARU -G v8-r`命令把 PKGBUILD 及相关文件获取到本地或者性能更优的编译机器。修改`PKGBUILD`中的arch为`riscv64`。参考[ arch 跨平台打包教程](https://github.com/felixonmars/archriscv-packages/wiki/archbuild-%E4%BD%BF%E7%94%A8%E5%8F%82%E8%80%83#%E8%B7%A8%E5%B9%B3%E5%8F%B0%E6%89%93%E5%8C%85)。进行第一次 `extra-riscv64-build -- -d "/tmp/cache:/var/cache/pacman/pkg"`,报错如下：

```
==> Starting prepare()...
  -> Fetching V8 code
/build/v8-r/src/depot_tools/vpython3: line 45: /build/v8-r/src/depot_tools/.cipd_bin/vpython3: No such file or directory
==> ERROR: A failure occurred in prepare().
    Aborting...
```  
在[谷歌源码站](https://chromium.googlesource.com/chromium/tools/depot_tools.git/+/refs/heads/main/vpython3)找到vpython3。第 45 行的上下文：
```
if [[ $(uname -s) = MINGW* || $(uname -s) = CYGWIN* ]]; then
  cmd.exe //c $0.bat "$@"
else
  exec "$base_dir/.cipd_bin/vpython3" "$@"
fi
```   
判断当前操作系统信息执行接下来的操作。`exec` 会用 `$base_dir/.cipd_bin/vpython3` 命令执行前面程序传入的解析后的 `@`。 但是报错是找不到 `.cipd_bin` 文件夹，`arch-chroot` 进入`build` 目录也不存在.cipd文件夹，在谷歌源码站的文件目录到处找了一圈，没有找到创建这个文件夹的过程，参照前文判断操作系统类型的命令，发现会先执行 `$0.bat` ，也就是 `vpython3.bat` ，这个文件在源码站有，简短的两句话：   
```
call "%~dp0\cipd_bin_setup.bat" > nul 2>&1
"%~dp0\.cipd_bin\vpython3.exe" %*
```   
调用 `cipd_bin_setup.bat` 批处理文件之后执行 `.cipd_bin\vpython3.exe` ，此处执行 `vpython3.exe` 和 判断条件之后的 `.cipd_bin/vpython3` 是相同的操作，所以它可能也是一个可执行程序，那么为什么会被局限在`.cipd`文件夹中呢。   
再来看看 `cipd_bin_setup.bat` 的代码：   
```bat
"%~dp0\cipd.bat" ensure -log-level warning -ensure-file "%~dp0\cipd_manifest.txt" -root "%~dp0\.cipd_bin"
```
调用 `cipd.bat` 根据 `cipd_manifest.txt` 的内容在 `.cipd_bin` 进行一些下载文件之类的操作。   
`cipd_bin_setup.sh` 的代码如下：   
```shell
function cipd_bin_setup {
    local MYPATH=$(dirname "${BASH_SOURCE[0]}")
    local ENSURE="$MYPATH/cipd_manifest.txt"
    local ROOT="$MYPATH/.cipd_bin"
    UNAME=`uname -s | tr '[:upper:]' '[:lower:]'`
    case $UNAME in
      cygwin*)
        ENSURE="$(cygpath -w $ENSURE)"
        ROOT="$(cygpath -w $ROOT)"
        ;;
    esac
    "$MYPATH/cipd" ensure \
        -log-level warning \
        -ensure-file "$ENSURE" \
        -root "$ROOT"
}
```   
并没有相关思路，于是 `sudo arch-chroot /var/lib/archbuild/extra-riscv64/` 在 build 目录下手动运行 `PKGBUILD` 中的命令， 发现 `cipd_bin_set_up.sh` 脚本根据 `cipd_manifest.txt` 文件获取到的 `PACKAGE ID` 并不是正确的，咨询了 v8 相关上游， 不能通过直接修改 `cipd_maniifest.txt` 的方法来使之正确，因为谷歌的 cipd 有自己的一套生成方法，但目前还不支持 `riscv` platform。实际上现在通过 `depot_tools` 官方工具构建v8是行不通的。但既然是 vpython 部分出了问题，为什么不绕过它直接用 python 呢？绕过 vpython 的PKGBUILD 的 [git](https://github.com/XYenChi/v8-r.git)。将 [vpython3 文件](https://chromium.googlesource.com/chromium/tools/depot_tools.git/+/refs/heads/main/vpython3)下载到文件夹中, 只留下参数解析相关的代码，然后使用 `python3` 执行。在 `PKGBUILD` 的 `PREPARE` 阶段将修改后的 `vpython3` 复制进未来的源文件中替代原有的 `vpython3` 即 `cp $srcdir/vpython3 $srcdir/depot_tools/vpython3` ，如此运行之后，还是会报很多 `package ID` 有问题的警告，fetch icu 结束之后，因为报 `package ID` 无法解析，结束 `fetch`，但还是之前说的，谷歌上游并没有更新生成最新 `package ID` 的支持，但 `icu` 也可以不用 `fetch` 下来的，直接使用系统的。将 `PKGBUILD` 中的 `yes | fetch v8` 改为 `（yes | fetch v8) || ：`，`fetch` 操作就会无论如何都继续进行下去。`PKGBUILD` 中的：   
```
  if [ -f third_party/icu/BUILD.gn.orig ]
   then
       msg2 "Restoring bundled ICU build files for syncing"
       $srcdir/v8/build/linux/unbundle/replace_gn_files.py --undo --system-libraries icu
  fi
```   
可以注释掉。在 `makedepends` 中加上`'python-six' 'python-httplib2' 'python-pyparsing' 'gn' 'ninja'`。`/v8/build/toolchain/linux/` 路径下的 `BUILD.gn` 复制出来，修改riscv toolchain 的 prefix 。原本 `PKGBUILD` 使用的 `gn` 前加上 `/usr/bin` 以使用编译机器安装的 `gn`，此时运行会因为 `use_lld=true` 这个参数出现动态链接问题，将其改为 `use_lld=false` 。原本`PKGBUILD` 使用 `ninja` 前面加上 `/usr/bin` 以使用编译机器安装的 `ninja` 。此时便可以打出了可以使用的 v8 软件包。 用 `PKGBUILD` 打包的 v8 并没有 benchmark 相关的代码，于是在谷歌官网上用 `depot_tools` fetch 下来。以下是测试结果。
#### Kraken
```
ai-astar-orig,38224.9,734.26,79
audio-beat-detection-orig,32087.8,1192.43,79
audio-dft-orig,30613.8,1424.52,79
audio-fft-orig,31178.3,1377.77,79
audio-oscillator-orig,25875.8,1708.79,79
imaging-gaussian-blur-orig,156007.7,4088.03,79
imaging-darkroom-orig,30422.7,875.37,79
imaging-desaturate-orig,56887.5,1964.78,79
json-parse-financial-orig,411.4,13.50,79
json-stringify-tinderbox-orig,355.7,15.57,79
stanford-crypto-aes-orig,9221.0,434.73,79
stanford-crypto-ccm-orig,5322.5,291.78,79
stanford-crypto-pbkdf2-orig,17789.8,897.05,79
stanford-crypto-sha256-iterative-orig,5660.6,295.99,79
Kraken,440059.5,15314.54,79
```   
#### Octane
```
RayTrace,138.7,8.72,9
EarleyBoyer,264.9,19.36,9
RegExp,113.4,5.22,9
Splay,296.3,8.92,9
SplayLatency,1455.6,21.20,9
NavierStokes,62.9,3.02,9
PdfJS,405.4,19.51,9
Mandreel,46.7,1.87,9
MandreelLatency,291.4,14.85,9
Gameboy,324.3,16.59,9
CodeLoad,2666.2,85.86,9
Box2D,150.2,8.63,9
zlib,92.6,4.42,9
Typescript,859.2,42.45,9
Richards,71.6,2.33,8
DeltaBlue,67.8,4.95,8
Crypto,58.1,2.30,8
Octane,203.3,15.48,8
```   
#### SunSpider
```
3d-morph-sunspider,448.8,4.43,99
3d-raytrace-sunspider,468.9,9.96,99
access-binary-trees-sunspider,114.3,1.64,99
access-fannkuch-sunspider,821.2,24.34,99
access-nbody-sunspider,590.1,7.19,99
access-nsieve-sunspider,300.1,4.18,99
bitops-3bit-bits-in-byte-sunspider,184.9,2.42,99
bitops-bits-in-byte-sunspider,308.9,4.84,99
bitops-bitwise-and-sunspider,363.6,13.93,99
bitops-nsieve-bits-sunspider,622.9,13.47,99
controlflow-recursive-sunspider,121.2,2.94,99
crypto-aes-sunspider,321.7,7.01,99
crypto-md5-sunspider,150.6,1.86,99
crypto-sha1-sunspider,155.7,3.78,99
date-format-tofte-sunspider,369.2,4.11,99
date-format-xparb-sunspider,182.8,2.59,99
math-cordic-sunspider,485.3,2.53,99
math-partial-sums-sunspider,354.8,7.82,99
math-spectral-norm-sunspider,242.6,1.59,99
regexp-dna-sunspider,288.7,1.26,99
string-base64-sunspider,190.0,5.57,99
string-fasta-sunspider,332.1,9.04,99
string-tagcloud-sunspider,279.9,4.14,99
string-unpack-code-sunspider,330.2,3.52,99
string-validate-input-sunspider,177.0,4.59,99
SunSpider,8694.4,154.47,99
```   
### SpiderMonkey
SpiderMonkey 是 firefox 的 JavaScript 引擎，firefox 在Arch Risc-v 上已经有了现成的包，只需要找到 Kraken, Octane 和 SunSpider 这三个benchmark的源码进行测试即可，Octane 和 SunSpider在 github 上有源码，Kraken 从v8的源代码里抠出来，但是其中使用了 `d8.file.execute()`的函数，由于笔者对 JavaScript 没有任何研究，不知道如何将对 d8 的调用修改为 spidermonkey 的，该函数的注释为 “load test data” ，所以参照从 github 上 clone 下来的 Octane 和 SunSpider 修改为`load()`以下是测试结果：   
#### Kraken
```
ai-astar-orig(RunTime): 68920 ms.
audio-beat-detection-orig(RunTime): 32697 ms.
audio-dft-orig(RunTime): 33860 ms.
audio-fft-orig(RunTime): 30655 ms.
audio-oscillator-orig(RunTime): 41767 ms.
imaging-gaussian-blur-orig(RunTime): 359305 ms.
imaging-darkroom-orig(RunTime): 47741 ms.
imaging-desaturate-orig(RunTime): 64943 ms.
json-parse-financial-orig(RunTime): 490 ms.
json-stringify-tinderbox-orig(RunTime): 254 ms.
stanford-crypto-aes-orig(RunTime): 11408 ms.
stanford-crypto-ccm-orig(RunTime): 8265 ms.
stanford-crypto-pbkdf2-orig(RunTime): 21507 ms.
stanford-crypto-sha256-iterative-orig(RunTime): 5959 ms.
```
#### Octane
```
Richards: 14.0
DeltaBlue: 14.1
Crypto: 30.8
RayTrace: 41.4
EarleyBoyer: 62.5
RegExp: 21.5
Splay: 92.9
SplayLatency: 533
NavierStokes: 66.2
PdfJS: 177
Mandreel: 16.9
MandreelLatency: 97.7
Gameboy: 133
CodeLoad: 2262
Box2D: 86.3
zlib: 66.7
Typescript: 295
----
Score (version 9): 78.6
```   
#### SunSpider
```
3d-cube: 739
3d-morph: 1179
3d-raytrace: 882
access-binary-trees: 705
access-fannkuch: 1477
access-nbody: 1117
access-nsieve: 712
bitops-3bit-bits-in-byte: 555
bitops-bits-in-byte: 855
bitops-bitwise-and: 3187
bitops-nsieve-bits: 685
controlflow-recursive: 785
crypto-aes: 711
crypto-md5: 551
crypto-sha1: 545
date-format-tofte: 852
date-format-xparb: 543
math-cordic: 974
math-partial-sums: 1314
math-spectral-norm: 681
regexp-dna: 376
string-base64: 568
string-fasta: 1352
string-tagcloud: 1085
string-unpack-code: 2446
string-validate-input: 2009
```