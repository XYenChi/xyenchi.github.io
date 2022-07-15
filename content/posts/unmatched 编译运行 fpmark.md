---
title: "Unmatched 编译运行 Fpmark"
date: 2022-07-11T14:22:10+08:00
draft: false
---
现有如下两块 unmatched 用来跑 benchmark, 一台叫 lemontea, 另一台叫 milk 。
   
lemontea 系统信息，clang 版本，gcc 版本如下：
```shell
[root@lemontea ~]# uname -a
Linux lemontea 5.18.3-arch1-1 #1 SMP PREEMPT Sun, 12 Jun 2022 18:42:25 +0000 riscv64 GNU/Linux
[root@lemontea ~]# clang -v
clang version 13.0.1
Target: riscv64-unknown-linux-gnu
Thread model: posix
InstalledDir: /usr/bin
Found candidate GCC installation: /usr/bin/../lib/gcc/riscv64-unknown-linux-gnu/12.1.0
Selected GCC installation: /usr/bin/../lib/gcc/riscv64-unknown-linux-gnu/12.1.0
[root@lemontea ~]# gcc -v
Using built-in specs.
COLLECT_GCC=gcc
COLLECT_LTO_WRAPPER=/usr/lib/gcc/riscv64-unknown-linux-gnu/12.1.0/lto-wrapper
Target: riscv64-unknown-linux-gnu
Configured with: /build/gcc/src/gcc/configure --enable-languages=c,c++,fortran,go,lto,objc,obj-c++ --enable-bootstrap --prefix=/usr --libdir=/usr/lib --libexecdir=/usr/lib --mandir=/usr/share/man --infodir=/usr/share/info --with-bugurl=https://bugs.archlinux.org/ --with-linker-hash-style=gnu --with-system-zlib --enable-__cxa_atexit --enable-cet=auto --enable-checking=release --enable-clocale=gnu --enable-default-pie --enable-default-ssp --enable-gnu-indirect-function --enable-gnu-unique-object --enable-linker-build-id --enable-lto --disable-multilib --enable-plugin --enable-shared --enable-threads=posix --disable-libssp --disable-libstdcxx-pch --disable-werror --enable-link-serialization=1
Thread model: posix
Supported LTO compression algorithms: zlib zstd
gcc version 12.1.0 (GCC)
```
milk 系统信息，clang 版本，gcc 版本如下：
```shell
[root@milk ~]# uname -a
Linux milk 5.18.3-arch1-1 #1 SMP PREEMPT Sun, 12 Jun 2022 18:42:25 +0000 riscv64 GNU/Linux
[root@milk ~]# clang -v
clang version 14.0.6
Target: riscv64-unknown-linux-gnu
Thread model: posix
InstalledDir: /usr/bin
Found candidate GCC installation: /usr/bin/../lib/gcc/riscv64-unknown-linux-gnu/12.1.0
Selected GCC installation: /usr/bin/../lib/gcc/riscv64-unknown-linux-gnu/12.1.0
[root@milk ~]# gcc -v
Using built-in specs.
COLLECT_GCC=gcc
COLLECT_LTO_WRAPPER=/usr/lib/gcc/riscv64-unknown-linux-gnu/12.1.0/lto-wrapper
Target: riscv64-unknown-linux-gnu
Configured with: /build/gcc/src/gcc/configure --enable-languages=c,c++,fortran,go,lto,objc,obj-c++ --enable-bootstrap --prefix=/usr --libdir=/usr/lib --libexecdir=/usr/lib --mandir=/usr/share/man --infodir=/usr/share/info --with-bugurl=https://bugs.archlinux.org/ --with-linker-hash-style=gnu --with-system-zlib --enable-__cxa_atexit --enable-cet=auto --enable-checking=release --enable-clocale=gnu --enable-default-pie --enable-default-ssp --enable-gnu-indirect-function --enable-gnu-unique-object --enable-linker-build-id --enable-lto --disable-multilib --enable-plugin --enable-shared --enable-threads=posix --disable-libssp --disable-libstdcxx-pch --disable-werror --enable-link-serialization=1
Thread model: posix
Supported LTO compression algorithms: zlib zstd
gcc version 12.1.0 (GCC)
```   
唯一的不同是clang版本，lemontea是13.0.1, milk是14.0.6。
### 前情提要：   

1.第一次运行报 `/util/perl/cert_mark.pl` 338行有除零错误。虽然发现是作者没有打分号的问题，但是加上分号还是会有测试项值为0的问题，加上一个判断 `$single==0` 以防万一。
```diff
diff --git a/util/perl/cert_mark.pl b/util/perl/cert_mark.pl
index 3e41e70..c818a56 100644
--- a/util/perl/cert_mark.pl
+++ b/util/perl/cert_mark.pl
@@ -335,7 +335,11 @@ sub print_multimarks {
                if ($single < 0 || $best < 0) {
                        print "$0: ERROR: Could not compute score for '$mark'\n";
                } else {
-                       printf "%-47s %10.2f %10.2f %10.2f\n", $mark, $best, $single, $best / $single
+                       if ($single == 0) {
+                               printf "%-47s %10.2f %10.2f \tinf\n", $mark, $best, $single;
+                       } else {
+                               printf "%-47s %10.2f %10.2f %10.2f\n", $mark, $best, $single, $best / $single;
+                       }
                }
        }
 }
```
2.前几次测试出现了某些项为0的情况，初步怀疑并修改了较为容易修改的perl文件数据精度，修改前精度为小数点后两位，修改后精度为小数点后8位。发生了变化的为 ray-1024x768at24s 此项，从原本的“0.00 0.00 n/a”变为“0.00894690 0.00224030 3.99361693“。
   
### fpmark的工作流程大致是：   
1.编译运行下列测试程序生成log文件   
```
atan-1k                  horner-mid-10k               linear_alg-sml-50x50     nnet_data1
atan-1k-sp               horner-mid-10k-sp            linear_alg-sml-50x50-sp  nnet-data1-sp
atan-1M                  horner-sml-1k                loops-all-big-100k       nnet_test
atan-1M-sp               horner-sml-1k-sp             loops-all-big-100k-sp    nnet_test-sp
atan-64k                 inner-product-big-100k       loops-all-mid-10k        radix2-big-64k
atan-64k-sp              inner-product-big-100k-sp    loops-all-mid-10k-sp     radix2-mid-8k
blacks-big-n5000v200     inner-product-mid-10k        loops-all-tiny           radix2-sml-2k
blacks-big-n5000v200-sp  inner-product-mid-10k-sp     loops-all-tiny-sp        ray-1024x768at24s
blacks-mid-n1000v40      inner-product-sml-1k         lu-big-2000x2_50         ray-320x240at8s
blacks-mid-n1000v40-sp   inner-product-sml-1k-sp      lu-big-2000x2_50-sp      ray-64x48at4s
blacks-sml-n500v20       linear_alg-big-1000x1000     lu-mid-200x2_50      
blacks-sml-n500v20-sp    linear_alg-big-1000x1000-sp  lu-mid-200x2_50-sp       xp1px-big-c10000n2000
horner-big-100k          linear_alg-mid-100x100       lu-sml-20x2_50           xp1px-mid-c1000n200
horner-big-100k-sp       linear_alg-mid-100x100-sp    lu-sml-20x2_50-sp        xp1px-sml-c100n20
```
2.Perl程序对log文件进行计算，生成分数表   

### 以不同参数编译运行fpmark记录
使用如下命令，用默认方式编译运行fpmark，XCMD='-c4' 指定四核编译：
```
make XCMD='-c4' certify-all
```
```
WORKLOAD RESULTS TABLE

                                                 MultiCore SingleCore
Workload Name                                     (iter/s)   (iter/s)    Scaling
----------------------------------------------- ---------- ---------- ----------
atan-1M                                         9.67117988 3.06184936 3.15860735
atan-1M-sp                                      16.05136437 5.01002004 3.20385233
atan-1k                                         15105.74018127 3786.44452859 3.98942598
atan-1k-sp                                      23148.14814815 5878.89476778 3.93750000
atan-64k                                        218.00741225 56.97356427 3.82646610
atan-64k-sp                                     346.38032560 87.10042679 3.97679252
blacks-big-n5000v200                            0.87573343 0.26504108 3.30414225
blacks-big-n5000v200-sp                         1.27713921 0.37596812 3.39693485
blacks-mid-n1000v40                             22.32142857 7.11743772 3.13616072
blacks-mid-n1000v40-sp                          32.36245955 9.84251969 3.28802589
blacks-sml-n500v20                              91.74311927 27.70083102 3.31192661
blacks-sml-n500v20-sp                           133.33333333 39.68253968 3.36000000
horner-big-100k                                 77.88161994 22.72210861 3.42757009
horner-big-100k-sp                              118.62396204 35.10003510 3.37959668
horner-mid-10k                                  1001.00100100 251.50905433 3.97997998
horner-mid-10k-sp                               1400.56022409 351.12359551 3.98879552
horner-sml-1k                                   9920.63492063 2486.32521134 3.99007937
horner-sml-1k-sp                                13623.97820163 3419.97264022 3.98365123
inner-product-big-100k                          4.71698113 2.25912120 2.08797170
inner-product-big-100k-sp                       11.04972376 4.07000407 2.71491713
inner-product-mid-10k                           93.76465073 25.87322122 3.62400375
inner-product-mid-10k-sp                        172.48814144 55.22573519 3.12332902
inner-product-sml-1k                            1315.78947368 526.87038988 2.49736842
inner-product-sml-1k-sp                         3460.20761246 952.38095238 3.63321799
linear_alg-big-1000x1000                        0.04487122 0.01851153 2.42396063
linear_alg-big-1000x1000-sp                     0.08103596 0.03873657 2.09197562
linear_alg-mid-100x100                          37.28560776 9.71628449 3.83743475
linear_alg-mid-100x100-sp                       50.25125628 13.06847883 3.84522613
linear_alg-sml-50x50                            320.92426187 80.89305938 3.96726573
linear_alg-sml-50x50-sp                         380.80731150 95.82215408 3.97410510
loops-all-big-100k                              0.03514099 0.01844848 1.90481763
loops-all-big-100k-sp                           0.05241804 0.02882243 1.81865443
loops-all-mid-10k                               0.82866519 0.30080797 2.75479799
loops-all-mid-10k-sp                            1.28014747 0.37028534 3.45719188
loops-all-tiny                                  972.76264591 246.91358025 3.93968872
loops-all-tiny-sp                               1253.13283208 314.46540881 3.98496241
lu-big-2000x2_50                                1.94174757 0.58288645 3.33126215
lu-big-2000x2_50-sp                             2.19973603 0.66203244 3.32270127
lu-mid-200x2_50                                 187.44142455 46.93513564 3.99362699
lu-mid-200x2_50-sp                              196.07843137 49.11108928 3.99254902
lu-sml-20x2_50                                  2115.95429539 529.10052910 3.99915362
lu-sml-20x2_50-sp                               2260.90888537 565.09945750 4.00090436
nnet-data1-sp                                   1782.53119430 448.63167340 3.97326203
nnet_data1                                      1234.56790123 308.45157310 4.00246914
nnet_test                                       2.03541624 0.61728395 3.29737431
nnet_test-sp                                    2.40847784 0.72134459 3.33887281
radix2-big-64k                                  108.44810758 39.56165684 2.74124282
radix2-mid-8k                                   1857.01021356 475.91852275 3.90194986
radix2-sml-2k                                   15344.48365812 3841.13082892 3.99478288
ray-1024x768at24s                               0.00894690 0.00224030 3.99361693
ray-320x240at8s                                 0.22830529 0.06841770 3.33693313
ray-64x48at4s                                   13.07873398 3.39789331 3.84907141
xp1px-big-c10000n2000                           0.21970779 0.06558022 3.35021429
xp1px-mid-c1000n200                             22.12389381 6.84931507 3.23008850
xp1px-sml-c100n20                               2770.08310249 688.23124570 4.02493075

MARK RESULTS TABLE

Mark Name                                        MultiCore SingleCore    Scaling
----------------------------------------------- ---------- ---------- ----------
cert_mark.pl: ERROR: Errors encountered in test lu-sml-20x2_50-sp, single run
cert_mark.pl: ERROR: Errors encountered in test lu-big-2000x2_50, single run
cert_mark.pl: ERROR: Errors encountered in test lu-mid-200x2_50, single run
cert_mark.pl: ERROR: Errors encountered in test lu-sml-20x2_50, single run
cert_mark.pl: ERROR: Errors encountered in test lu-mid-200x2_50-sp, single run
cert_mark.pl: ERROR: Errors encountered in test lu-big-2000x2_50-sp, single run
cert_mark.pl: ERROR: Errors encountered in test lu-sml-20x2_50-sp, best run
cert_mark.pl: ERROR: Errors encountered in test lu-big-2000x2_50, best run
cert_mark.pl: ERROR: Errors encountered in test lu-mid-200x2_50, best run
cert_mark.pl: ERROR: Errors encountered in test lu-sml-20x2_50, best run
cert_mark.pl: ERROR: Errors encountered in test lu-mid-200x2_50-sp, best run
cert_mark.pl: ERROR: Errors encountered in test lu-big-2000x2_50-sp, best run
cert_mark.pl: ERROR: Could not compute score for 'FPMark'
cert_mark.pl: ERROR: Errors encountered in test lu-sml-20x2_50, single run
cert_mark.pl: ERROR: Errors encountered in test lu-sml-20x2_50, best run
cert_mark.pl: ERROR: Could not compute score for 'FPv1.0. DP Small Dataset'
cert_mark.pl: ERROR: Errors encountered in test lu-mid-200x2_50, single run
cert_mark.pl: ERROR: Errors encountered in test lu-mid-200x2_50, best run
cert_mark.pl: ERROR: Could not compute score for 'FPv1.1. DP Medium Dataset'
cert_mark.pl: ERROR: Errors encountered in test lu-big-2000x2_50, single run
cert_mark.pl: ERROR: Errors encountered in test lu-big-2000x2_50, best run
cert_mark.pl: ERROR: Could not compute score for 'FPv1.2. DP Big Dataset'
cert_mark.pl: ERROR: Errors encountered in test lu-sml-20x2_50-sp, single run
cert_mark.pl: ERROR: Errors encountered in test lu-sml-20x2_50-sp, best run
cert_mark.pl: ERROR: Could not compute score for 'FPv1.3. SP Small Dataset'
cert_mark.pl: ERROR: Errors encountered in test lu-mid-200x2_50-sp, single run
cert_mark.pl: ERROR: Errors encountered in test lu-mid-200x2_50-sp, best run
cert_mark.pl: ERROR: Could not compute score for 'FPv1.4. SP Medium Dataset'
cert_mark.pl: ERROR: Errors encountered in test lu-big-2000x2_50-sp, single run
cert_mark.pl: ERROR: Errors encountered in test lu-big-2000x2_50-sp, best run
cert_mark.pl: ERROR: Could not compute score for 'FPv1.5. SP Big Dataset'
cert_mark.pl: ERROR: Errors encountered in test lu-mid-200x2_50, single run
cert_mark.pl: ERROR: Errors encountered in test lu-big-2000x2_50, single run
cert_mark.pl: ERROR: Errors encountered in test lu-sml-20x2_50, single run
cert_mark.pl: ERROR: Errors encountered in test lu-mid-200x2_50, best run
cert_mark.pl: ERROR: Errors encountered in test lu-big-2000x2_50, best run
cert_mark.pl: ERROR: Errors encountered in test lu-sml-20x2_50, best run
cert_mark.pl: ERROR: Could not compute score for 'FPv1.D. DP Mark'
cert_mark.pl: ERROR: Errors encountered in test lu-mid-200x2_50-sp, single run
cert_mark.pl: ERROR: Errors encountered in test lu-big-2000x2_50-sp, single run
cert_mark.pl: ERROR: Errors encountered in test lu-sml-20x2_50-sp, single run
cert_mark.pl: ERROR: Errors encountered in test lu-mid-200x2_50-sp, best run
cert_mark.pl: ERROR: Errors encountered in test lu-big-2000x2_50-sp, best run
cert_mark.pl: ERROR: Errors encountered in test lu-sml-20x2_50-sp, best run
cert_mark.pl: ERROR: Could not compute score for 'FPv1.S. SP Mark'
cert_mark.pl: ERROR: Errors encountered in test lu-sml-20x2_50-sp, single run
cert_mark.pl: ERROR: Errors encountered in test lu-sml-20x2_50-sp, best run
cert_mark.pl: ERROR: Could not compute score for 'MicroFPMark'
``` 

找到报对应错误的 /util/perl/cert_mark.pl 文件中的对应代码：
```perl
 elsif ($g_scores{$v}{$mode}{'errors'} != 0) 
        print "$0: ERROR: Errors encountered in test $v, $mode run\n";
        $errors = 1;

``` 
本段 `recurse_geomean` 函数功能为迭代计算几何平均数   
`$v` 传入的值： 在 `%g_mark_definitions` 表中查询 `$factor` 值的项，此处为 `lu-*`。   
`$mode` 传入的值： `$single` 或 `$best`。   
会报 `ERROR: Errors encountered in test` 说明查询 `$g_scores` 时，对应的 `errors` 项不为0。
找到改变 `$errors` 的代码：   
```perl
if ($mode eq "verification") {
			$most_recent_error{$name} = $fails;
		}
```
以及
```perl
$g_scores{$name}{$scoring}{'ips'} = $ips;
my $errors = $most_recent_error{$name};
	if not defined $errors;
	$g_scores{$name}{$scoring}{'errors'} = $errors;
```
说明 log 文件中有测试项的 `$fails` 为1, 找到对应lu的log，发现 Results for verification run 该项的 Fails 全为1：
```
#UID            Suite Name                                     Ctx Wrk Fails       t(s)       Iter     Iter/s  Codesize   Datasize

#Results for verification run started at 22194:08:30:50 XCMD=
1210558733        MLT lu-big-2000x2_50                           1   1     1 1.79400000          1 0.55741360   1392507       3680
#Results for performance runs started at 22194:08:30:52 XCMD=
1210558733        MLT lu-big-2000x2_50                           1   1     0 17.45100000         10 0.57303306   1392507       3680
1210558733        MLT lu-big-2000x2_50                           1   1     0 17.46200000         10 0.57267209   1392507       3680
1210558733        MLT lu-big-2000x2_50                           1   1     0 17.42500000         10 0.57388809   1392507       3680
#Median for final result lu-big-2000x2_50/
1210558733        MLT lu-big-2000x2_50                           1   1     0 17.45100000         10 0.57303306   1392507       3680 median single
#UID            Suite Name                                     Ctx Wrk Fails       t(s)       Iter     Iter/s  Codesize   Datasize
#Results for verification run started at 22194:08:31:47 XCMD=
1204991441        MLT lu-big-2000x2_50-sp                        1   1     1 1.60600000          1 0.62266501   1396552       3656
#Results for performance runs started at 22194:08:31:49 XCMD=
1204991441        MLT lu-big-2000x2_50-sp                        1   1     0 15.47600000         10 0.64616180   1396552       3656
1204991441        MLT lu-big-2000x2_50-sp                        1   1     0 15.48300000         10 0.64586966   1396552       3656
1204991441        MLT lu-big-2000x2_50-sp                        1   1     0 15.47600000         10 0.64616180   1396552       3656
#Median for final result lu-big-2000x2_50-sp/
1204991441        MLT lu-big-2000x2_50-sp                        1   1     0 15.47600000         10 0.64616180   1396552       3656 median single
#UID            Suite Name                                     Ctx Wrk Fails       t(s)       Iter     Iter/s  Codesize   Datasize
#Results for verification run started at 22194:08:32:38 XCMD=
208607378         MLT lu-mid-200x2_50                            1   1     1 0.02500000          1 40.00000000   1392503       3680
#Results for performance runs started at 22194:08:32:38 XCMD=
208607378         MLT lu-mid-200x2_50                            1   1     0 21.89500000       1000 45.67252797   1392503       3680
208607378         MLT lu-mid-200x2_50                            1   1     0 21.88900000       1000 45.68504728   1392503       3680
208607378         MLT lu-mid-200x2_50                            1   1     0 21.91000000       1000 45.64125970   1392503       3680
#Median for final result lu-mid-200x2_50/
208607378         MLT lu-mid-200x2_50                            1   1     0 21.89500000       1000 45.67252797   1392503       3680 median single
#UID            Suite Name                                     Ctx Wrk Fails       t(s)       Iter     Iter/s  Codesize   Datasize
#Results for verification run started at 22194:08:33:46 XCMD=
97815375          MLT lu-mid-200x2_50-sp                         1   1     1 0.02400000          1 41.66666667   1396548       3656
#Results for performance runs started at 22194:08:33:46 XCMD=
97815375          MLT lu-mid-200x2_50-sp                         1   1     0 21.13400000       1000 47.31711933   1396548       3656
97815375          MLT lu-mid-200x2_50-sp                         1   1     0 21.12100000       1000 47.34624308   1396548       3656
97815375          MLT lu-mid-200x2_50-sp                         1   1     0 21.15000000       1000 47.28132388   1396548       3656
#Median for final result lu-mid-200x2_50-sp/
97815375          MLT lu-mid-200x2_50-sp                         1   1     0 21.13400000       1000 47.31711933   1396548       3656 median single
#UID            Suite Name                                     Ctx Wrk Fails       t(s)       Iter     Iter/s  Codesize   Datasize
#Results for verification run started at 22194:08:34:52 XCMD=
429137479         MLT lu-sml-20x2_50                             1   1     1 0.00200000          1 500.00000000   1392495       3688
#Results for performance runs started at 22194:08:34:53 XCMD=
429137479         MLT lu-sml-20x2_50                             1   1     0 19.44300000      10000 514.32392121   1392495       3688
429137479         MLT lu-sml-20x2_50                             1   1     0 19.43200000      10000 514.61506793   1392495       3688
429137479         MLT lu-sml-20x2_50                             1   1     0 19.45800000      10000 513.92743345   1392495       3688
#Median for final result lu-sml-20x2_50/
429137479         MLT lu-sml-20x2_50                             1   1     0 19.44300000      10000 514.32392121   1392495       3688 median single
#UID            Suite Name                                     Ctx Wrk Fails       t(s)       Iter     Iter/s  Codesize   Datasize
#Results for verification run started at 22194:08:35:53 XCMD=
1203890958        MLT lu-sml-20x2_50-sp                          1   1     1 0.00200000          1 500.00000000   1396544       3656
#Results for performance runs started at 22194:08:35:54 XCMD=
1203890958        MLT lu-sml-20x2_50-sp                          1   1     0 18.38400000      10000 543.95126197   1396544       3656
1203890958        MLT lu-sml-20x2_50-sp                          1   1     0 18.38100000      10000 544.04004135   1396544       3656
1203890958        MLT lu-sml-20x2_50-sp                          1   1     0 18.38700000      10000 543.86251156   1396544       3656
#Median for final result lu-sml-20x2_50-sp/
1203890958        MLT lu-sml-20x2_50-sp                          1   1     0 18.38400000      10000 543.95126197   1396544       3656 median single
```
由于已知在 x86 上交叉编译后移植到 unmatched 上运行可以成功出结果，所以此处怀疑是 riscv-linux64-gnu-gcc 的问题。为了将问题定位在编译器上，尝试使用 clang 来编译运行 fpmark。在 /util/make/ 中复制 gcc.mak， 另存为 clang.mak, 并将原有 gcc 的配置改成 clang 和相关的LLVM工具链，diff 如下。
```diff
< #  File: util/make/clang.mak
< #     LLVM Tool Definitions, Host Compile and Run
---
> #  File: util/make/gcc.mak
> #     GCC Tool Definitions, Host Compile and Run
39c39
< CC            = $(TOOLS)/bin/clang
---
> CC            = $(TOOLS)/bin/gcc
50c50
< AS            = $(TOOLS)/bin/llvm-as
---
> AS            = $(TOOLS)/bin/as
54,55c54,55
< LD            = $(TOOLS)/bin/clang
< LDPP  = $(TOOLS)/bin/clang++
---
> LD            = $(TOOLS)/bin/gcc
> LDPP  = $(TOOLS)/bin/g++
61c61
< AR            = $(TOOLS)/bin/llvm-ar
---
> AR            = $(TOOLS)/bin/ar
```
然后用下面的命令在 `lemontea` 编译运行，发现可以通过并出分。
```
make toolchain=clang XCMD='-c4' certify-all
```   
```
WORKLOAD RESULTS TABLE

                                                 MultiCore SingleCore
Workload Name                                     (iter/s)   (iter/s)    Scaling
----------------------------------------------- ---------- ---------- ----------
atan-1M                                         9.86193294 3.20410125 3.07790927
atan-1M-sp                                      16.05136437 5.16795866 3.10593900
atan-1k                                         15625.00000000 3923.10710082 3.98281250
atan-1k-sp                                      24096.38554217 6056.93519079 3.97831325
atan-64k                                        221.92632046 58.44876965 3.79693742
atan-64k-sp                                     359.71223022 88.60535176 4.05971223
blacks-big-n5000v200                            0.85404390 0.25453712 3.35528233
blacks-big-n5000v200-sp                         1.23107226 0.36564408 3.36685954
blacks-mid-n1000v40                             21.73913043 6.65778961 3.26521739
blacks-mid-n1000v40-sp                          32.36245955 9.30232558 3.47896440
blacks-sml-n500v20                              87.71929825 26.52519894 3.30701754
blacks-sml-n500v20-sp                           121.95121951 39.21568627 3.10975610
horner-big-100k                                 47.86979416 13.16309069 3.63666826
horner-big-100k-sp                              70.57163020 19.45146859 3.62808751
horner-mid-10k                                  554.63117027 139.54786492 3.97448697
horner-mid-10k-sp                               774.59333850 193.87359442 3.99535244
horner-sml-1k                                   5494.50549451 1383.89150291 3.97032967
horner-sml-1k-sp                                7651.10941086 1914.24196018 3.99693956
inner-product-big-100k                          4.51161741 1.56445557 2.88382585
inner-product-big-100k-sp                       9.97008973 2.47616689 4.02642074
inner-product-mid-10k                           83.50730689 22.53394175 3.70584551
inner-product-mid-10k-sp                        145.08523758 40.57618178 3.57562568
inner-product-sml-1k                            1145.47537228 428.26552463 2.67468499
inner-product-sml-1k-sp                         2512.56281407 673.40067340 3.73115578
linear_alg-big-1000x1000                        0.03922030 0.01651419 2.37494543
linear_alg-big-1000x1000-sp                     0.06956135 0.03194929 2.17724244
linear_alg-mid-100x100                          27.24795640 7.18081287 3.79455041
linear_alg-mid-100x100-sp                       38.13882532 9.92063492 3.84439359
linear_alg-sml-50x50                            233.64485981 58.71301080 3.97943925
linear_alg-sml-50x50-sp                         293.77203290 73.64854912 3.98883666
loops-all-big-100k                              0.03294372 0.01691607 1.94748071
loops-all-big-100k-sp                           0.04904702 0.02602567 1.88456320
loops-all-mid-10k                               0.76624830 0.26448729 2.89710821
loops-all-mid-10k-sp                            1.15561513 0.33888426 3.41005844
loops-all-tiny                                  713.26676177 179.59770115 3.97146933
loops-all-tiny-sp                               917.43119266 229.99080037 3.98899083
lu-big-2000x2_50                                1.21506683 0.36532349 3.32600247
lu-big-2000x2_50-sp                             1.45687646 0.43715847 3.33260490
lu-mid-200x2_50                                 122.39902081 30.70215836 3.98665851
lu-mid-200x2_50-sp                              136.05442177 34.18803419 3.97959184
lu-sml-20x2_50                                  1376.84152554 344.67307759 3.99463032
lu-sml-20x2_50-sp                               1577.53588894 395.55397334 3.98816848
nnet-data1-sp                                   1362.39782016 345.06556246 3.94822888
nnet_data1                                      1025.64102564 257.33401956 3.98564103
nnet_test                                       1.68947457 0.50880228 3.32049332
nnet_test-sp                                    1.83755972 0.55215063 3.32800439
radix2-big-64k                                  95.77626664 34.50655625 2.77559621
radix2-mid-8k                                   1565.19017061 395.38193895 3.95867898
radix2-sml-2k                                   10192.64091326 2557.87185062 3.98481297
ray-1024x768at24s                               0.00749033 0.00187651 3.99162808
ray-320x240at8s                                 0.19111323 0.05735063 3.33236496
ray-64x48at4s                                   10.96972356 2.84884052 3.85059237
xp1px-big-c10000n2000                           0.21477663 0.06374908 3.36909380
xp1px-mid-c1000n200                             21.73913043 6.77048070 3.21086957
xp1px-sml-c100n20                               2673.79679144 694.92703266 3.84759358

MARK RESULTS TABLE

Mark Name                                        MultiCore SingleCore    Scaling
----------------------------------------------- ---------- ---------- ----------
FPMark                                          4105.87406511 1184.54552948 3.46620198
FPv1.0. DP Small Dataset                        955.61807387 254.65795873 3.75255530
FPv1.1. DP Medium Dataset                       27.29842116 7.69152449 3.54915611
FPv1.2. DP Big Dataset                          0.92118438 0.30547377 3.01559241
FPv1.3. SP Small Dataset                        1546.52098102 403.87376625 3.82921871
FPv1.4. SP Medium Dataset                       44.14215642 11.93425978 3.69877623
FPv1.5. SP Big Dataset                          1.83142747 0.61449165 2.98039439
FPv1.D. DP Mark                                 32.13685141 9.34692868 3.43822581
FPv1.S. SP Mark                                 57.73558984 16.46998052 3.50550444
MicroFPMark                                     1546.52098102 403.87376625 3.82921871
m
```   

说明问题出在编译器上，通过查看 `gcc64.mak` 文件发现会以 `-O2` 参数默认编译，排除优化问题还不太复杂，于是先使用 `-O0` 参数编译运行了一遍，发现可以成功出分，又使用 `-O1` 参数，也可以成功出分。说明问题在 `-O1` 与 `-O2` 中。重新编译运行之前需要 `make clean`一下，删除上一次的 build 产物，不然会直接跳过 build 。
man 一下 gcc 看看 -O2 是在干啥：
```
       -O2 Optimize even more.  GCC performs nearly all supported optimizations that do not involve a space-speed
           tradeoff.  As compared to -O, this option increases both compilation time and the performance of the
           generated code.

           -O2 turns on all optimization flags specified by -O1.  It also turns on the following optimization flags:

           -falign-functions  -falign-jumps -falign-labels  -falign-loops -fcaller-saves -fcode-hoisting
           -fcrossjumping -fcse-follow-jumps  -fcse-skip-blocks -fdelete-null-pointer-checks -fdevirtualize
           -fdevirtualize-speculatively -fexpensive-optimizations -ffinite-loops -fgcse  -fgcse-lm
           -fhoist-adjacent-loads -finline-functions -finline-small-functions -findirect-inlining -fipa-bit-cp
           -fipa-cp  -fipa-icf -fipa-ra  -fipa-sra  -fipa-vrp -fisolate-erroneous-paths-dereference -flra-remat
           -foptimize-sibling-calls -foptimize-strlen -fpartial-inlining -fpeephole2 -freorder-blocks-algorithm=stc
           -freorder-blocks-and-partition  -freorder-functions -frerun-cse-after-loop -fschedule-insns
           -fschedule-insns2 -fsched-interblock  -fsched-spec -fstore-merging -fstrict-aliasing -fthread-jumps
           -ftree-builtin-call-dce -ftree-loop-vectorize -ftree-pre -ftree-slp-vectorize -ftree-switch-conversion
           -ftree-tail-merge -ftree-vrp -fvect-cost-model=very-cheap
```   
是不是觉得参数有点多，即使使用二分法，每跑一次用三四个小时也令人痛苦。但是看看 Make.mak 这个文件，就会找到一次编译运行验证一个程序的神秘代码：
```make
#Target: wcertify-%
#	Build, run and collect results for certification procedure on specific workloads
wcertify-%: wbuild-%
```
再看看编译器的参数是如何传入的,如果不传入 DDB 或者 DDN 程序会以默认 `CFLAGS = $(COMPILER_FLAGS) $(COMPILER_DEFS) $(PLATFORM_DEFS) $(PACK_OPTS)` 运行，其中 `COMPILER_FLAGS = -O2 $(CDEFN)NDEBUG $(CDEFN)HOST_EXAMPLE_CODE=1 -std=gnu99 `：   
```make
COMPILER_FLAGS = -O2 $(CDEFN)NDEBUG $(CDEFN)HOST_EXAMPLE_CODE=1 -std=gnu99 
COMPILER_NOOPT = -g -O0 $(CDEFN)NDEBUG $(CDEFN)HOST_EXAMPLE_CODE=1 
PACK_OPTS = 

ifdef DDN
 CFLAGS = $(COMPILER_NOOPT) $(COMPILER_DEFS) $(PLATFORM_DEFS) $(PACK_OPTS) 
else
 CFLAGS = $(COMPILER_FLAGS) $(COMPILER_DEFS) $(PLATFORM_DEFS) $(PACK_OPTS)
endif
```   
所以加入 `DDN=1` 关闭默认优化，使用 `PACK_OPT` 分别传入 `-O0` `-O1` `-O2` 进行对比。   
1. PACK_OPTS='-O0'
```
make DDN=1 PACK_OPTS='-O0' wcertify-lu-sml-20x2_50
```   
```
-  Info: Starting Run...
-- Workload:lu-sml-20x2_50=429137479
-- lu-sml-20x2_50:time(ns)=12
-- lu-sml-20x2_50:contexts=1
-- lu-sml-20x2_50:iterations=1
-- lu-sml-20x2_50:time(secs)=   0.012
-- lu-sml-20x2_50:secs/workload=   0.012
-- lu-sml-20x2_50:workloads/sec= 83.3333
Info: This run was executed with verification turned on! For performance results, use -v0.
-- matrix01[5]:UID=10000
-- matrix01[5]:fails=0
-- matrix01[5]:time(ticks)=11
-- matrix01[5]:count=1
-- matrix01[5]:repeats=1
-- matrix01[5]:v1=49
-- matrix01[5]:v2=0
-- matrix01[5]:v3=0
-- matrix01[5]:v4=0
-- matrix01[5]:f1=0.000000e+00
-- matrix01[5]:f2=0.000000e+00
-- matrix01[5]:f3=0.000000e+00
-- matrix01[5]:f4=0.000000e+00
-- matrix01[5]:secs/repeat=   0.011
-- matrix01[5]:repeats/sec= 90.9091
-- matrix01[5]:time(secs)=   0.011
-- matrix01[5]:secs/item=   0.011
-- matrix01[5]:items/sec= 90.9091
-- Items:total(ticks)=11
-- Items:total(secs)=   0.011
-- Done:lu-sml-20x2_50=429137479
```   
2. PACK_OPTS='-O1'   
```
make DDN=1 PACK_OPTS='-O1' wcertify-lu-sml-20x2_50
```   
```
-  Info: Starting Run...
-- Workload:lu-sml-20x2_50=429137479
-- lu-sml-20x2_50:time(ns)=4
-- lu-sml-20x2_50:contexts=1
-- lu-sml-20x2_50:iterations=1
-- lu-sml-20x2_50:time(secs)=   0.004
-- lu-sml-20x2_50:secs/workload=   0.004
-- lu-sml-20x2_50:workloads/sec=     250
Info: This run was executed with verification turned on! For performance results, use -v0.
-- matrix01[5]:UID=10000
-- matrix01[5]:fails=0
-- matrix01[5]:time(ticks)=4
-- matrix01[5]:count=1
-- matrix01[5]:repeats=1
-- matrix01[5]:v1=49
-- matrix01[5]:v2=0
-- matrix01[5]:v3=0
-- matrix01[5]:v4=0
-- matrix01[5]:f1=0.000000e+00
-- matrix01[5]:f2=0.000000e+00
-- matrix01[5]:f3=0.000000e+00
-- matrix01[5]:f4=0.000000e+00
-- matrix01[5]:secs/repeat=   0.004
-- matrix01[5]:repeats/sec=     250
-- matrix01[5]:time(secs)=   0.004
-- matrix01[5]:secs/item=   0.004
-- matrix01[5]:items/sec=     250
-- Items:total(ticks)=4
-- Items:total(secs)=   0.004
-- Done:lu-sml-20x2_50=429137479
```   
3.   PACK_OPTS='-O2'  
```
make DDN=1 PACK_OPTS='-O2' wcertify-lu-sml-20x2_50
```    
```
-  Info: Starting Run...
LU decomposition ERROR: a:0,44
-- Workload:lu-sml-20x2_50=429137479
-- lu-sml-20x2_50:time(ns)=3
-- lu-sml-20x2_50:ERRORS=1
-- lu-sml-20x2_50:contexts=1
-- lu-sml-20x2_50:iterations=1
-- lu-sml-20x2_50:time(secs)=   0.003
-- lu-sml-20x2_50:secs/workload=   0.003
-- lu-sml-20x2_50:workloads/sec= 333.333
Info: This run was executed with verification turned on! For performance results, use -v0.
-- matrix01[5]:UID=10000
-- matrix01[5]:fails=1
-- matrix01[5]:time(ticks)=2
-- matrix01[5]:count=1
-- matrix01[5]:repeats=1
-- matrix01[5]:v1=49
-- matrix01[5]:v2=0
-- matrix01[5]:v3=0
-- matrix01[5]:v4=0
-- matrix01[5]:f1=0.000000e+00
-- matrix01[5]:f2=0.000000e+00
-- matrix01[5]:f3=0.000000e+00
-- matrix01[5]:f4=0.000000e+00
-- matrix01[5]:secs/repeat=   0.002
-- matrix01[5]:repeats/sec=     500
-- matrix01[5]:time(secs)=   0.002
-- matrix01[5]:secs/item=   0.002
-- matrix01[5]:items/sec=     500
-- Items:total(ticks)=2
-- Items:total(secs)=   0.002
-- accbits:min=0
-- accbits:max=52
-- accbits:avg=44
-- Done:lu-sml-20x2_50=429137479
```   
接下来看看 `-O1` 和 `-O2` 之间的区别   
先跑参数的前五行：   
```
make DDN=1 PACK_OPTS='-O1 -falign-functions  -falign-jumps -falign-labels  -falign-loops -fcaller-saves -fcode-hoisting -fcrossjumping -fcse-follow-jumps  -fcse-skip-blocks -fdelete-null-pointer-checks -fdevirtualize -fdevirtualize-speculatively -fexpensive-optimizations -ffinite-loops -fgcse  -fgcse-lm -fhoist-adjacent-loads -finline-functions -finline-small-functions -findirect-inlining -fipa-bit-cp -fipa-cp  -fipa-icf -fipa-ra  -fipa-sra  -fipa-vrp -fisolate-erroneous-paths-dereference -flra-remat' wcertify-lu-sml-20x2_50
```   
```
-  Info: Starting Run...
LU decomposition ERROR: a:0,44
-- Workload:lu-sml-20x2_50=429137479
-- lu-sml-20x2_50:time(ns)=3
-- lu-sml-20x2_50:ERRORS=1
-- lu-sml-20x2_50:contexts=1
-- lu-sml-20x2_50:iterations=1
-- lu-sml-20x2_50:time(secs)=   0.003
-- lu-sml-20x2_50:secs/workload=   0.003
-- lu-sml-20x2_50:workloads/sec= 333.333
Info: This run was executed with verification turned on! For performance results, use -v0.
-- matrix01[5]:UID=10000
-- matrix01[5]:fails=1
-- matrix01[5]:time(ticks)=2
-- matrix01[5]:count=1
-- matrix01[5]:repeats=1
-- matrix01[5]:v1=49
-- matrix01[5]:v2=0
-- matrix01[5]:v3=0
-- matrix01[5]:v4=0
-- matrix01[5]:f1=0.000000e+00
-- matrix01[5]:f2=0.000000e+00
-- matrix01[5]:f3=0.000000e+00
-- matrix01[5]:f4=0.000000e+00
-- matrix01[5]:secs/repeat=   0.002
-- matrix01[5]:repeats/sec=     500
-- matrix01[5]:time(secs)=   0.002
-- matrix01[5]:secs/item=   0.002
-- matrix01[5]:items/sec=     500
-- Items:total(ticks)=2
-- Items:total(secs)=   0.002
-- accbits:min=0
-- accbits:max=52
-- accbits:avg=44
-- Done:lu-sml-20x2_50=429137479
```   
出现问题了，但是不能排除只有前五行有，所以跑一下后五行：
```
make DDN=1 PACK_OPTS='-O1 -foptimize-sibling-calls -foptimize-strlen -fpartial-inlining -fpeephole2 -freorder-blocks-algorithm=stc -freorder-blocks-and-partition  -freorder-functions -frerun-cse-after-loop -fschedule-insns -fschedule-insns2 -fsched-interblock  -fsched-spec -fstore-merging -fstrict-aliasing -fthread-jumps -ftree-builtin-call-dce -ftree-loop-vectorize -ftree-pre -ftree-slp-vectorize -ftree-switch-conversion -ftree-tail-merge -ftree-vrp -fvect-cost-model=very-cheap' wcertify-lu-sml-20x2_50
```   
```
-  Info: Starting Run...
-- Workload:lu-sml-20x2_50=429137479
-- lu-sml-20x2_50:time(ns)=3
-- lu-sml-20x2_50:contexts=1
-- lu-sml-20x2_50:iterations=1
-- lu-sml-20x2_50:time(secs)=   0.003
-- lu-sml-20x2_50:secs/workload=   0.003
-- lu-sml-20x2_50:workloads/sec= 333.333
Info: This run was executed with verification turned on! For performance results, use -v0.
-- matrix01[5]:UID=10000
-- matrix01[5]:fails=0
-- matrix01[5]:time(ticks)=3
-- matrix01[5]:count=1
-- matrix01[5]:repeats=1
-- matrix01[5]:v1=49
-- matrix01[5]:v2=0
-- matrix01[5]:v3=0
-- matrix01[5]:v4=0
-- matrix01[5]:f1=0.000000e+00
-- matrix01[5]:f2=0.000000e+00
-- matrix01[5]:f3=0.000000e+00
-- matrix01[5]:f4=0.000000e+00
-- matrix01[5]:secs/repeat=   0.003
-- matrix01[5]:repeats/sec= 333.333
-- matrix01[5]:time(secs)=   0.003
-- matrix01[5]:secs/item=   0.003
-- matrix01[5]:items/sec= 333.333
-- Items:total(ticks)=3
-- Items:total(secs)=   0.003
-- Done:lu-sml-20x2_50=429137479
```   
看结果问题出现在前五行的优化参数，继续二分法：   
```
make DDN=1 PACK_OPTS='-O1 -falign-functions  -falign-jumps -falign-labels  -falign-loops -fcaller-saves -fcode-hoisting -fcrossjumping -fcse-follow-jumps  -fcse-skip-blocks -fdelete-null-pointer-checks -fdevirtualize -fdevirtualize-speculatively -fexpensive-optimizations -ffinite-loops -fgcse  -fgcse-lm -fhoist-adjacent-loads -finline-functions -finline-small-functions -findirect-inlining -fipa-bit-cp -fipa-cp  -fipa-icf -fipa-ra  -fipa-sra  -fipa-vrp -fisolate-erroneous-paths-dereference -flra-remat' wcertify-lu-sml-20x2_50
```   
```
-  Info: Starting Run...
LU decomposition ERROR: a:0,44
-- Workload:lu-sml-20x2_50=429137479
-- lu-sml-20x2_50:time(ns)=3
-- lu-sml-20x2_50:ERRORS=1
-- lu-sml-20x2_50:contexts=1
-- lu-sml-20x2_50:iterations=1
-- lu-sml-20x2_50:time(secs)=   0.003
-- lu-sml-20x2_50:secs/workload=   0.003
-- lu-sml-20x2_50:workloads/sec= 333.333
Info: This run was executed with verification turned on! For performance results, use -v0.
-- matrix01[5]:UID=10000
-- matrix01[5]:fails=1
-- matrix01[5]:time(ticks)=2
-- matrix01[5]:count=1
-- matrix01[5]:repeats=1
-- matrix01[5]:v1=49
-- matrix01[5]:v2=0
-- matrix01[5]:v3=0
-- matrix01[5]:v4=0
-- matrix01[5]:f1=0.000000e+00
-- matrix01[5]:f2=0.000000e+00
-- matrix01[5]:f3=0.000000e+00
-- matrix01[5]:f4=0.000000e+00
-- matrix01[5]:secs/repeat=   0.002
-- matrix01[5]:repeats/sec=     500
-- matrix01[5]:time(secs)=   0.002
-- matrix01[5]:secs/item=   0.002
-- matrix01[5]:items/sec=     500
-- Items:total(ticks)=2
-- Items:total(secs)=   0.002
-- accbits:min=0
-- accbits:max=52
-- accbits:avg=44
-- Done:lu-sml-20x2_50=429137479
```   

```
make DDN=1 PACK_OPTS='-O1 -falign-functions  -falign-jumps -falign-labels  -falign-loops -fcaller-saves -fcode-hoisting -fcrossjumping -fcse-follow-jumps  -fcse-skip-blocks -fdelete-null-pointer-checks -fdevirtualize -fdevirtualize-speculatively -fexpensive-optimizations -ffinite-loops' wcertify-lu-sml-20x2_50
```   
```
-  Info: Starting Run...
LU decomposition ERROR: a:0,44
-- Workload:lu-sml-20x2_50=429137479
-- lu-sml-20x2_50:time(ns)=3
-- lu-sml-20x2_50:ERRORS=1
-- lu-sml-20x2_50:contexts=1
-- lu-sml-20x2_50:iterations=1
-- lu-sml-20x2_50:time(secs)=   0.003
-- lu-sml-20x2_50:secs/workload=   0.003
-- lu-sml-20x2_50:workloads/sec= 333.333
Info: This run was executed with verification turned on! For performance results, use -v0.
-- matrix01[5]:UID=10000
-- matrix01[5]:fails=1
-- matrix01[5]:time(ticks)=2
-- matrix01[5]:count=1
-- matrix01[5]:repeats=1
-- matrix01[5]:v1=49
-- matrix01[5]:v2=0
-- matrix01[5]:v3=0
-- matrix01[5]:v4=0
-- matrix01[5]:f1=0.000000e+00
-- matrix01[5]:f2=0.000000e+00
-- matrix01[5]:f3=0.000000e+00
-- matrix01[5]:f4=0.000000e+00
-- matrix01[5]:secs/repeat=   0.002
-- matrix01[5]:repeats/sec=     500
-- matrix01[5]:time(secs)=   0.002
-- matrix01[5]:secs/item=   0.002
-- matrix01[5]:items/sec=     500
-- Items:total(ticks)=2
-- Items:total(secs)=   0.002
-- accbits:min=0
-- accbits:max=52
-- accbits:avg=44
-- Done:lu-sml-20x2_50=429137479
```

```
make DDN=1 PACK_OPTS='-O1 -fgcse  -fgcse-lm -fhoist-adjacent-loads -finline-functions -finline-small-functions -findirect-inlining -fipa-bit-cp -fipa-cp  -fipa-icf -fipa-ra  -fipa-sra  -fipa-vrp -fisolate-erroneous-paths-dereference -flra-remat' wcertify-lu-sml-20x2_50
```
```
-  Info: Starting Run...
-- Workload:lu-sml-20x2_50=429137479
-- lu-sml-20x2_50:time(ns)=4
-- lu-sml-20x2_50:contexts=1
-- lu-sml-20x2_50:iterations=1
-- lu-sml-20x2_50:time(secs)=   0.004
-- lu-sml-20x2_50:secs/workload=   0.004
-- lu-sml-20x2_50:workloads/sec=     250
Info: This run was executed with verification turned on! For performance results, use -v0.
-- matrix01[5]:UID=10000
-- matrix01[5]:fails=0
-- matrix01[5]:time(ticks)=3
-- matrix01[5]:count=1
-- matrix01[5]:repeats=1
-- matrix01[5]:v1=49
-- matrix01[5]:v2=0
-- matrix01[5]:v3=0
-- matrix01[5]:v4=0
-- matrix01[5]:f1=0.000000e+00
-- matrix01[5]:f2=0.000000e+00
-- matrix01[5]:f3=0.000000e+00
-- matrix01[5]:f4=0.000000e+00
-- matrix01[5]:secs/repeat=   0.003
-- matrix01[5]:repeats/sec= 333.333
-- matrix01[5]:time(secs)=   0.003
-- matrix01[5]:secs/item=   0.003
-- matrix01[5]:items/sec= 333.333
-- Items:total(ticks)=3
-- Items:total(secs)=   0.003
-- Done:lu-sml-20x2_50=429137479
```   
```
make DDN=1 PACK_OPTS='-O1 -falign-functions  -falign-jumps -falign-labels  -falign-loops -fcaller-saves -fcode-hoisting -fcrossjumping' wcertify-lu-sml-20x2_50
```   
```
-  Info: Starting Run...
-- Workload:lu-sml-20x2_50=429137479
-- lu-sml-20x2_50:time(ns)=4
-- lu-sml-20x2_50:contexts=1
-- lu-sml-20x2_50:iterations=1
-- lu-sml-20x2_50:time(secs)=   0.004
-- lu-sml-20x2_50:secs/workload=   0.004
-- lu-sml-20x2_50:workloads/sec=     250
Info: This run was executed with verification turned on! For performance results, use -v0.
-- matrix01[5]:UID=10000
-- matrix01[5]:fails=0
-- matrix01[5]:time(ticks)=4
-- matrix01[5]:count=1
-- matrix01[5]:repeats=1
-- matrix01[5]:v1=49
-- matrix01[5]:v2=0
-- matrix01[5]:v3=0
-- matrix01[5]:v4=0
-- matrix01[5]:f1=0.000000e+00
-- matrix01[5]:f2=0.000000e+00
-- matrix01[5]:f3=0.000000e+00
-- matrix01[5]:f4=0.000000e+00
-- matrix01[5]:secs/repeat=   0.004
-- matrix01[5]:repeats/sec=     250
-- matrix01[5]:time(secs)=   0.004
-- matrix01[5]:secs/item=   0.004
-- matrix01[5]:items/sec=     250
-- Items:total(ticks)=4
-- Items:total(secs)=   0.004
-- Done:lu-sml-20x2_50=429137479
```   

```
make DDN=1 PACK_OPTS='-O1 -fcse-skip-blocks -fdelete-null-pointer-checks -fdevirtualize -fdevirtualize-speculatively -fexpensive-optimizations -ffinite-loops' wcertify-lu-sml-20x2_50
```
```
-  Info: Starting Run...
LU decomposition ERROR: a:0,44
-- Workload:lu-sml-20x2_50=429137479
-- lu-sml-20x2_50:time(ns)=3
-- lu-sml-20x2_50:ERRORS=1
-- lu-sml-20x2_50:contexts=1
-- lu-sml-20x2_50:iterations=1
-- lu-sml-20x2_50:time(secs)=   0.003
-- lu-sml-20x2_50:secs/workload=   0.003
-- lu-sml-20x2_50:workloads/sec= 333.333
Info: This run was executed with verification turned on! For performance results, use -v0.
-- matrix01[5]:UID=10000
-- matrix01[5]:fails=1
-- matrix01[5]:time(ticks)=2
-- matrix01[5]:count=1
-- matrix01[5]:repeats=1
-- matrix01[5]:v1=49
-- matrix01[5]:v2=0
-- matrix01[5]:v3=0
-- matrix01[5]:v4=0
-- matrix01[5]:f1=0.000000e+00
-- matrix01[5]:f2=0.000000e+00
-- matrix01[5]:f3=0.000000e+00
-- matrix01[5]:f4=0.000000e+00
-- matrix01[5]:secs/repeat=   0.002
-- matrix01[5]:repeats/sec=     500
-- matrix01[5]:time(secs)=   0.002
-- matrix01[5]:secs/item=   0.002
-- matrix01[5]:items/sec=     500
-- Items:total(ticks)=2
-- Items:total(secs)=   0.002
-- accbits:min=0
-- accbits:max=52
-- accbits:avg=44
-- Done:lu-sml-20x2_50=429137479
```   

```
make DDN=1 PACK_OPTS='-O1 -fcse-skip-blocks -fdelete-null-pointer-checks -fdevirtualize' wcertify-lu-sml-20x2_50
```   
```
-  Info: Starting Run...
-- Workload:lu-sml-20x2_50=429137479
-- lu-sml-20x2_50:time(ns)=4
-- lu-sml-20x2_50:contexts=1
-- lu-sml-20x2_50:iterations=1
-- lu-sml-20x2_50:time(secs)=   0.004
-- lu-sml-20x2_50:secs/workload=   0.004
-- lu-sml-20x2_50:workloads/sec=     250
Info: This run was executed with verification turned on! For performance results, use -v0.
-- matrix01[5]:UID=10000
-- matrix01[5]:fails=0
-- matrix01[5]:time(ticks)=4
-- matrix01[5]:count=1
-- matrix01[5]:repeats=1
-- matrix01[5]:v1=49
-- matrix01[5]:v2=0
-- matrix01[5]:v3=0
-- matrix01[5]:v4=0
-- matrix01[5]:f1=0.000000e+00
-- matrix01[5]:f2=0.000000e+00
-- matrix01[5]:f3=0.000000e+00
-- matrix01[5]:f4=0.000000e+00
-- matrix01[5]:secs/repeat=   0.004
-- matrix01[5]:repeats/sec=     250
-- matrix01[5]:time(secs)=   0.004
-- matrix01[5]:secs/item=   0.004
-- matrix01[5]:items/sec=     250
-- Items:total(ticks)=4
-- Items:total(secs)=   0.004
-- Done:lu-sml-20x2_50=429137479
```   

```
make DDN=1 PACK_OPTS='-O1 -fdevirtualize-speculatively -fexpensive-optimizations -ffinite-loops' wcertify-lu-sml-20x2_50
```   
```
-  Info: Starting Run...
LU decomposition ERROR: a:0,44
-- Workload:lu-sml-20x2_50=429137479
-- lu-sml-20x2_50:time(ns)=3
-- lu-sml-20x2_50:ERRORS=1
-- lu-sml-20x2_50:contexts=1
-- lu-sml-20x2_50:iterations=1
-- lu-sml-20x2_50:time(secs)=   0.003
-- lu-sml-20x2_50:secs/workload=   0.003
-- lu-sml-20x2_50:workloads/sec= 333.333
Info: This run was executed with verification turned on! For performance results, use -v0.
-- matrix01[5]:UID=10000
-- matrix01[5]:fails=1
-- matrix01[5]:time(ticks)=2
-- matrix01[5]:count=1
-- matrix01[5]:repeats=1
-- matrix01[5]:v1=49
-- matrix01[5]:v2=0
-- matrix01[5]:v3=0
-- matrix01[5]:v4=0
-- matrix01[5]:f1=0.000000e+00
-- matrix01[5]:f2=0.000000e+00
-- matrix01[5]:f3=0.000000e+00
-- matrix01[5]:f4=0.000000e+00
-- matrix01[5]:secs/repeat=   0.002
-- matrix01[5]:repeats/sec=     500
-- matrix01[5]:time(secs)=   0.002
-- matrix01[5]:secs/item=   0.002
-- matrix01[5]:items/sec=     500
-- Items:total(ticks)=2
-- Items:total(secs)=   0.002
-- accbits:min=0
-- accbits:max=52
-- accbits:avg=44
-- Done:lu-sml-20x2_50=429137479
```      

```
make DDN=1 PACK_OPTS='-O1 -ffinite-loops' wcertify-lu-sml-20x2_50
```   
```
-  Info: Starting Run...
-- Workload:lu-sml-20x2_50=429137479
-- lu-sml-20x2_50:time(ns)=4
-- lu-sml-20x2_50:contexts=1
-- lu-sml-20x2_50:iterations=1
-- lu-sml-20x2_50:time(secs)=   0.004
-- lu-sml-20x2_50:secs/workload=   0.004
-- lu-sml-20x2_50:workloads/sec=     250
Info: This run was executed with verification turned on! For performance results, use -v0.
-- matrix01[5]:UID=10000
-- matrix01[5]:fails=0
-- matrix01[5]:time(ticks)=4
-- matrix01[5]:count=1
-- matrix01[5]:repeats=1
-- matrix01[5]:v1=49
-- matrix01[5]:v2=0
-- matrix01[5]:v3=0
-- matrix01[5]:v4=0
-- matrix01[5]:f1=0.000000e+00
-- matrix01[5]:f2=0.000000e+00
-- matrix01[5]:f3=0.000000e+00
-- matrix01[5]:f4=0.000000e+00
-- matrix01[5]:secs/repeat=   0.004
-- matrix01[5]:repeats/sec=     250
-- matrix01[5]:time(secs)=   0.004
-- matrix01[5]:secs/item=   0.004
-- matrix01[5]:items/sec=     250
-- Items:total(ticks)=4
-- Items:total(secs)=   0.004
-- Done:lu-sml-20x2_50=429137479
```   

```
make DDN=1 PACK_OPTS='-O1 -fdevirtualize-speculatively' wcertify-lu-sml-20x2_50
```   
```
-  Info: Starting Run...
-- Workload:lu-sml-20x2_50=429137479
-- lu-sml-20x2_50:time(ns)=4
-- lu-sml-20x2_50:contexts=1
-- lu-sml-20x2_50:iterations=1
-- lu-sml-20x2_50:time(secs)=   0.004
-- lu-sml-20x2_50:secs/workload=   0.004
-- lu-sml-20x2_50:workloads/sec=     250
Info: This run was executed with verification turned on! For performance results, use -v0.
-- matrix01[5]:UID=10000
-- matrix01[5]:fails=0
-- matrix01[5]:time(ticks)=4
-- matrix01[5]:count=1
-- matrix01[5]:repeats=1
-- matrix01[5]:v1=49
-- matrix01[5]:v2=0
-- matrix01[5]:v3=0
-- matrix01[5]:v4=0
-- matrix01[5]:f1=0.000000e+00
-- matrix01[5]:f2=0.000000e+00
-- matrix01[5]:f3=0.000000e+00
-- matrix01[5]:f4=0.000000e+00
-- matrix01[5]:secs/repeat=   0.004
-- matrix01[5]:repeats/sec=     250
-- matrix01[5]:time(secs)=   0.004
-- matrix01[5]:secs/item=   0.004
-- matrix01[5]:items/sec=     250
-- Items:total(ticks)=4
-- Items:total(secs)=   0.004
-- Done:lu-sml-20x2_50=429137479
```   

```
make DDN=1 PACK_OPTS='-O1 -fexpensive-optimizations' wcertify-lu-sml-20x2_50
```    
```
-  Info: Starting Run...
LU decomposition ERROR: a:0,44
-- Workload:lu-sml-20x2_50=429137479
-- lu-sml-20x2_50:time(ns)=3
-- lu-sml-20x2_50:ERRORS=1
-- lu-sml-20x2_50:contexts=1
-- lu-sml-20x2_50:iterations=1
-- lu-sml-20x2_50:time(secs)=   0.003
-- lu-sml-20x2_50:secs/workload=   0.003
-- lu-sml-20x2_50:workloads/sec= 333.333
Info: This run was executed with verification turned on! For performance results, use -v0.
-- matrix01[5]:UID=10000
-- matrix01[5]:fails=1
-- matrix01[5]:time(ticks)=3
-- matrix01[5]:count=1
-- matrix01[5]:repeats=1
-- matrix01[5]:v1=49
-- matrix01[5]:v2=0
-- matrix01[5]:v3=0
-- matrix01[5]:v4=0
-- matrix01[5]:f1=0.000000e+00
-- matrix01[5]:f2=0.000000e+00
-- matrix01[5]:f3=0.000000e+00
-- matrix01[5]:f4=0.000000e+00
-- matrix01[5]:secs/repeat=   0.003
-- matrix01[5]:repeats/sec= 333.333
-- matrix01[5]:time(secs)=   0.003
-- matrix01[5]:secs/item=   0.003
-- matrix01[5]:items/sec= 333.333
-- Items:total(ticks)=3
-- Items:total(secs)=   0.003
-- accbits:min=0
-- accbits:max=52
-- accbits:avg=44
-- Done:lu-sml-20x2_50=429137479
```   
现在就可以确定问题出在 gcc 优化的 `-fexpensive-optimization` 参数上。

### gcc优化问题解决方案：

```shell
make PACK_OPTS=-fno-expensive-optimizations XCMD='-c4' certify-all
```   

```
WORKLOAD RESULTS TABLE

                                                 MultiCore SingleCore
Workload Name                                     (iter/s)   (iter/s)    Scaling
----------------------------------------------- ---------- ---------- ----------
atan-1M                                         8.18330606 2.60213375 3.14484452
atan-1M-sp                                      12.31527094 3.84763371 3.20073891
atan-1k                                         12315.27093596 3087.37264588 3.98891626
atan-1k-sp                                      17301.03806228 4407.22785368 3.92560554
atan-64k                                        177.02248186 46.31130459 3.82244645
atan-64k-sp                                     260.62027626 68.75214850 3.79072192
blacks-big-n5000v200                            0.84452327 0.25670646 3.28984035
blacks-big-n5000v200-sp                         1.24579544 0.35936321 3.46667496
blacks-mid-n1000v40                             21.32196162 6.89655172 3.09168444
blacks-mid-n1000v40-sp                          31.05590062 9.45179584 3.28571429
blacks-sml-n500v20                              84.74576271 27.62430939 3.06779661
blacks-sml-n500v20-sp                           128.20512821 38.02281369 3.37179487
horner-big-100k                                 47.93863854 13.17523057 3.63854266
horner-big-100k-sp                              70.47216350 19.58480219 3.59830867
horner-mid-10k                                  557.10306407 139.72334777 3.98718663
horner-mid-10k-sp                               779.42322681 195.57989439 3.98519096
horner-sml-1k                                   5506.60792952 1385.23341183 3.97522026
horner-sml-1k-sp                                7680.49155146 1924.18703098 3.99155146
inner-product-big-100k                          4.60935699 2.06270627 2.23461627
inner-product-big-100k-sp                       10.17293998 3.82775120 2.65768057
inner-product-mid-10k                           85.16074090 23.95639935 3.55482223
inner-product-mid-10k-sp                        152.78838808 46.24812117 3.30366692
inner-product-sml-1k                            1173.70892019 443.65572316 2.64553991
inner-product-sml-1k-sp                         2557.54475703 716.84587814 3.56777494
linear_alg-big-1000x1000                        0.03923815 0.01624759 2.41501355
linear_alg-big-1000x1000-sp                     0.06809855 0.03154276 2.15892807
linear_alg-mid-100x100                          26.96871629 7.02641934 3.83818770
linear_alg-mid-100x100-sp                       36.33720930 9.45358291 3.84375000
linear_alg-sml-50x50                            226.03978300 56.77302146 3.98146474
linear_alg-sml-50x50-sp                         281.69014085 70.61149555 3.98929577
loops-all-big-100k                              0.03398471 0.01761711 1.92907406
loops-all-big-100k-sp                           0.05137321 0.02788265 1.84247946
loops-all-mid-10k                               0.82263903 0.29038534 2.83292204
loops-all-mid-10k-sp                            1.26304090 0.36520075 3.45848386
loops-all-tiny                                  848.89643463 213.58393849 3.97453311
loops-all-tiny-sp                               1098.90109890 277.16186253 3.96483516
lu-big-2000x2_50                                1.23365408 0.36960378 3.33777452
lu-big-2000x2_50-sp                             1.48301943 0.44577185 3.32685752
lu-mid-200x2_50                                 125.31328321 31.42973882 3.98709273
lu-mid-200x2_50-sp                              140.82523588 35.31696980 3.98746655
lu-sml-20x2_50                                  1405.08641281 351.86488388 3.99325559
lu-sml-20x2_50-sp                               1628.13415825 408.76389797 3.98306740
nnet-data1-sp                                   1522.07001522 382.40917782 3.98021309
nnet_data1                                      1094.09190372 273.89756231 3.99452954
nnet_test                                       1.80310133 0.54594093 3.30274070
nnet_test-sp                                    2.03583062 0.61357222 3.31799673
radix2-big-64k                                  106.11205433 38.96053298 2.72357810
radix2-mid-8k                                   1858.04533631 473.68670361 3.92251951
radix2-sml-2k                                   15179.11353977 3803.58297516 3.99074074
ray-1024x768at24s                               0.00849057 0.00212491 3.99573158
ray-320x240at8s                                 0.21629104 0.06494983 3.33012481
ray-64x48at4s                                   12.39157373 3.22268772 3.84510533
xp1px-big-c10000n2000                           0.21855535 0.06411818 3.40863309
xp1px-mid-c1000n200                             21.88183807 6.41436818 3.41137856
xp1px-sml-c100n20                               2673.79679144 696.86411150 3.83689840

MARK RESULTS TABLE

Mark Name                                        MultiCore SingleCore    Scaling
----------------------------------------------- ---------- ---------- ----------
FPMark                                          4147.02977385 1217.31202679 3.40671059
FPv1.0. DP Small Dataset                        1000.06368457 268.51665082 3.72440101
FPv1.1. DP Medium Dataset                       27.87460548 7.89095927 3.53247362
FPv1.2. DP Big Dataset                          0.93203344 0.31683771 2.94167459
FPv1.3. SP Small Dataset                        1550.62082138 403.81883253 3.83989229
FPv1.4. SP Medium Dataset                       43.44110782 12.03370930 3.60995157
FPv1.5. SP Big Dataset                          1.78201307 0.63281484 2.81601023
FPv1.D. DP Mark                                 32.99873946 9.71311985 3.39733680
FPv1.S. SP Mark                                 56.99147840 16.66517179 3.41979543
MicroFPMark                                     1550.62082138 403.81883253 3.83989229
```
### 疑问   

查阅了与 `-fexpensive-optimazion` 这个参数的资料，但是我没有看懂是什么意思，可能还需要时间变强，不如放在这等以后想起来再说。
```
       -fexpensive-optimizations
           Perform a number of minor optimizations that are relatively expensive.

           Enabled at levels -O2, -O3, -Os.
```

milk 这台 unmatched 是后面启用的，又正好遇到了 clang13 升级为 clang14 ，clang14 也出现了和 gcc -O2 一样的LU分解错误，但是 clang 究竟优化了什么还需要进一步研究。
