---
title: "RVV intrinsic的实现分析"
url: "/rvv_intrinsic_analysis/"
draft: false
---   
虽然标题叫分析，但现在已经变成了大量注释内容整合的形状，可能得改名为笔记。   
既然我第一行就写明了，那么便不是标题党。   
之所以不能叫分析，当然是自己回头看一遍都没看懂在写啥。   
### 包含的文件
[iterator.md](https://github.com/gcc-mirror/gcc/blob/master/gcc/config/riscv/iterator.md)   
[constraint.md](https://github.com/gcc-mirror/gcc/blob/master/gcc/config/riscv/constraint.md)   
[riscv-c.cc](https://github.com/gcc-mirror/gcc/blob/master/gcc/config/riscv/riscv-c.cc)   
[riscv-protos.h](https://github.com/gcc-mirror/gcc/blob/master/gcc/config/riscv/riscv-protos.h)   
[riscv-vector-builtins-bases.cc](https://github.com/gcc-mirror/gcc/blob/master/gcc/config/riscv/riscv-vector-builtins-bases.cc)   
[riscv-vector-builtins-bases.h](https://github.com/gcc-mirror/gcc/blob/master/gcc/config/riscv/riscv-vector-builtins-bases.h)   
[riscv-vector-builtins-functions.def](https://github.com/gcc-mirror/gcc/blob/master/gcc/config/riscv/riscv-vector-builtins-functions.def)   
[riscv-vector-builtins-shapes.cc](https://github.com/gcc-mirror/gcc/blob/master/gcc/config/riscv/riscv-vector-builtins-shapes.cc)   
[riscv-vector-builtins-shapes.h](https://github.com/gcc-mirror/gcc/blob/master/gcc/config/riscv/riscv-vector-builtins-shapes.h)   
[riscv-vector-builtins-types.def](https://github.com/gcc-mirror/gcc/blob/master/gcc/config/riscv/riscv-vector-builtins-types.def)    
[riscv-vector-builtins.cc](https://github.com/gcc-mirror/gcc/blob/master/gcc/config/riscv/riscv-vector-builtins.cc)   
[riscv-vector-builtins.h](https://github.com/gcc-mirror/gcc/blob/master/gcc/config/riscv/riscv-vector-builtins.h)   
[riscv.md](https://github.com/gcc-mirror/gcc/blob/master/gcc/config/riscv/riscv.md)   
[vector-iterators.md](https://github.com/gcc-mirror/gcc/blob/master/gcc/config/riscv/vector-iterators.md)   
[vector.md](https://github.com/gcc-mirror/gcc/blob/master/gcc/config/riscv/vector.md)   

### Acronyms   
QI: 8 bits   
HI: 16 bits   
SI: 32 bits   
DI: 64 bits   
把一个word（即 32 bit 当成计量单位）四分之一（quarter）、二分之一（half）、双倍（double）。   

#### iterator.md   
使用lisp声明通用寄存器位宽定义迭代器（iterator）名称      

#### constraint.md   
在 match_operand 中，可以指定操作数约束(operand  constraint)。   
约束(constraint)对断言(predicate)所允许的操作数进行更详细的描述。   
1. 约束条件可以定义操作数是否可以使用寄存器以及使用何种寄存器。   
2. 说明操作数是否可以是一个内存引用以及其地址类型。   
3. 描述该操作数是都可以是一个立即数常量(immediate)以及其可能的值。   
* GCC 中的约束(constraint)使用字符串(string)表示。   
以下是常见用法（部分夹带RISC-V私货方便自己查询）：   
***\>***:   memory operand, autoincrement addressing type, including preincrement and postincrement.   
***f***:   floating-point register   
***g***:   general register, memory or integer immediate constant.   
***i***:   Integer immediate operand, sign constant when and after compilering.   
***j***:   SIBCALL_REGS   
***l***:   JALR_REGS   
***n***:   known value integer immediate operand   
***p***:   memory address operand   
***x***:   all operand   
***I***:   12-bit integer signed immediate
***J***:   integer zero   
***K***:   5-bit unsigned immediate for CSR access instructions      
***L***:   U-type 20-bit signed immediate      
***Ds3***:   1, 2 or 3 immediate   
***DsS***:   31 immediate   
***DsD***:   63 immediate   
***DbS***:   
***DnS***:   
***D03***:   0, 1, 2 or 3 immediate   
***DsA***:   0 - 10 immediate   
***G***:   
***A***:   An address that is held in a general-purpose register.   
***S***:   A constraint that matches an absolute symbolic address.   
***U***:   A PLT-indirect call address.   
***T***:   
***vr***:   vector register      
***vd***:   vector register except mask register   
***vm***:   vector mask register   
***vp***:   poly int   
***vu***:   undefined vector value   
***vi***:   vector 5-bit signed immediate   
***vj***:   vector negated 5-bit signed immediate   
***vk***:   vector 5-bit unsigned immediate   
***Wc0***:   vector of immediate all zeros   
***Wc1***:   vector of immediate all ones   
***Wb1***:   BOOL vector of {...,0,...0,1}   
***Wdm***:   Vector duplicate memory operand   
***th_f_fmv***:   floating-point register for XTheadFmv   
***th_r_fmv***:   integer register for XTheadFmv   
***vmWc1***:   
vector mask register + a vector of immediate all ones   
***rK***:   
register operand using general register + 5-bit unsigned immediate for CSR access instructiosn
* 约束修饰字符 (Constraint Modifier Characters)   
***=***:操作数只写   
***+***:操作可读可写   
***&***:在某些约束选择(constraint alternative)中，该操作数是前面某个clobber的操作数，作为指令的输入操作数，该操作数在指令结束之前它的值已经被修改，因此，该操作数可能不在原来使用的寄存器或内存地址中存储      
***%***:可交换，该操作数及其之后的操作数可以进行交换   
eg. 操作数1的约束为 '%0'，表示与操作数0的约束相同。   
***#***:直到逗号的所有字符在进行约束处理时将被忽略，这些字符只对寄存器选择起作用   
***\*:*** 直到逗号的所有字符在进行约束处理是将被忽略，这些字符在寄存器选择是也将被忽略。   
#### riscv-c.cc
riscv intrinsic 相关。   
#### riscv-vector-builtins-bases.h
 包含所有向量操作的namespace，声明外部的常量指针。

#### riscv-vector-builtins-bases.cc
对 RISC-V v extension 中的指令对应的 intrinsic function 包含的元素进行声明。比如操作数类型、舍入形式、mask 类型等。   

实现 vsetvl 和 vsetvlmax 

实现运算的操作看起来都是根据操作数类型转到对应的rtl.

 #### riscv-vector-builtins-functions.def   
 速记 DEF_RVV_TYPE:   
 NAME: "vint32m1_t"   
 NCHARS: the length of ABI-name, ABI名的长度。"__rvv_int32m1_t" 的长度是15。   
 ABI_NAME: "__rvv_int32m1_t"   
 SCALAR_TYPE: 
 写 RVV intrinsic function 的名称，是否mask, 操作数符号和类型之类的。   

 #### riscv-vector-builtins-shapes.cc   
 定义函数 shape NAME, 指向类`<NAME>_def`实例。   
 存在 rvv 0.7 的类定义，但是继承自 `misc_def` 结构体，`misc_def` 结构体继承自 `build_base` 结构体， `build_base` 结构体继承自 `function_shape` 类。   

 ##### function_shape 类   
 写intrinsic function的格式
 #### riscv-vector-builtins-shapes.h
 在 riscv_vector 的命名空间里声明 shapes 的命名空间
 #### riscv-vector-builtins-types.def
 定义数据类型的宏
 #### riscv-vector-builtins.cc
 看起来是在写intrinsic格式。
 #### riscv-vector-builtins.h
 记录了 RVV intrinsic function 的命名方式：   
 - the base name ("vadd", etc.): 一般是 RISC-V 指令前面加个 'v'，除法指令的操作数顺序是反的，不要问我是怎么发现的（划掉。   
 - the operand suffix ("_vv", "_vx", etc.): 表明指令操作数类型，'v'表示向量，'x'表示标量。   
 - the type suffix ("_i32m1", "_i32mf2", etc.): 数字按照先后顺序表示 sew 和 lmul。   
 - the predication suffix ("_tamu", "_tumu", etc.): t表示tail，m表示mask，a表示agnostic，u表示undisturbed。   
 记录实现过程中名称代表的含义：   
 - function_base represents the base name.   
 - operand_type_index can be used as an index to get operand suffix.   
 - rvv_op_info can be used as an index to get argument suffix.   
 - predication_type_index can be used as an index to get predication suffix.   

 overloaded functions 移除了一些可以根据参数类型推测出来的后缀。   
 function_builder 类提供了一些辅助函数来添加 intrinsic function。    
 function_shape 类描述了指令如何在语言级别呈现。决定了 C/C++ overload 函数如何被编译器在语言级别识别；指定每个函数在语言级别呈现的的参数类型和返回类型。   
 
 riscv_vector 这个命名空间包括：   
 1.描述函数做什么的标识和读函数参数返回结果   
 2.定义用来识别RVV intrinsic需要的拓展的位值的宏   
 3.枚举 RVV 操作类型   

 声明 intrinsic 用到的数据类型，后缀的结构体。  
 function_base 类的定义   
 function_checker 类的定义   
 function_shape 类的定义   
 machine mode   
 *规定 intrinsic 特殊要求的bit表示*   

 ##### 用到的 rtx 种类   
 use_exact_ins   
 use_contiguous_load_insn
 use_contiguous_store_insn   
 use_compare_insn   
 use_ternop_ins   
 use_widen_ternop_insn   
 use_scalar_move_insn   
 generate_insn   
 ##### machine mode
 vector mode   
 index mode   
 arg mode   
 mask mode   
 ret mode   
 #### riscv.md
 写 vector 相关的 rtl   
 ##### attribute   
 has_vtype_op   
 在 gcc/config/riscv/riscv-vsetvl.cc 里定义的 bool 值。判断RVV指令是否会用到 VTYPE 全局状态寄存器。   
 `(define_attr "has_vtype_op" "false,true"` 表示 rtl 里面的 has_vtype_op 可以取到 false 或者 true。
 #### vector-iterators.md
 define_mode_iterator rtl   
 define_code_attr
 #### vector.md
 * 属性(Attribute)定义：   
`has_vtype_op`   
`has_vl_op`   
`sew`   
`lmul`   
`ratio`:sew/lmul   
`merge_op_idx`: "The index of operand[] to get the merge op."   
`vl_op_idx`: "The index of operand[] to get the avl op."   
`ta`: tail agnostic   
`ma`: mask agnostic   
`avl_type`   
`vxrm_mode`: fix-point. rnu,rne,rdn,rod,none   
`frm_mode`: float-point.   
 * 指令模板(Insn Pattern)定义：
`vlmax_avl`   
`vxrmsi`   
`fsrmsi_backup`   
`fsrmsi_restore`   
