---
title: "RVV intrinsic的实现分析"
date: 2023-07-17T12:59:38+08:00
url: "/rvv_intrinsic_analysis/"
draft: false
---
### 包含的文件
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

#### constraint.md   
在 match_operand 中，可以指定操作数约束(operand  constraint)。   
约束(constraint)对断言(predicate)所允许的操作数进行更详细的描述。   
1. 约束条件可以定义操作数是否可以使用寄存器以及使用何种寄存器。   
2. 说明操作数是否可以是一个内存引用以及其地址类型。   
3. 描述该操作数是都可以是一个立即数常量(immediate)以及其可能的值。   
GCC 中的约束(constraint)使用字符串(string)表示。   
\>:   memory operand, autoincrement addressing type, including preincrement and postincrement.   
f:   floating-point register   
g:   general register, memory or integer immediate constant.   
i:   Integer immediate operand, sign constant when and after compilering.   
j:   SIBCALL_REGS   
l:   JALR_REGS
n:   known value integer immediate operand
p:   memory address operand   
x:   all operand   
I:   12-bit integer signed immediate
J:   integer zero   
K:   5-bit unsigned immediate for CSR access instructions      
L:   U-type 20-bit signed immediate      
Ds3:   
DsS:   
DsD:   
DbS:   
DnS:   
D03:   
DsA:   
G:   
A:   
S:   
U:   
T:   
vr:   vector register      
vd:   vector register except mask register
vm:   vector mask register   
vp:   poly int
vu:   undefined vector value
vi:   vector 5-bit signed immediate
vj:   vector negated 5-bit signed immediate   
vk:   vector 5-bit unsigned immediate   
Wc0:   vector of immediate all zeros   
Wc1:   vector of immediate all ones   
Wb1:   BOOL vector of {...,0,...0,1}   
Wdm:   Vector duplicate memory operand   
th_f_fmv:   floating-point register for XTheadFmv   
th_r_fmv:   integer register for XTheadFmv   
vmWc1:   
vector mask register + a vector of immediate all ones   
rK:   
register operand using general register + 5-bit unsigned immediate for CSR access instructiosn
 
#### riscv-c.cc
`#define builtin_define(TXT) cpp_define (pfile, TXT)` 定义 `builtin_define` 的宏，将其展开为 `cpp_define(pfile, TXT)`。其中 pfile 是指向文件的指针。
 riscv_ext_version 函数，可以传一个最大值和一个最小值，返回 1000000倍的最大值和 1000倍的最小值。
 riscv_cpu_cpp_builtins 函数，传入 pfile 指针，

 #### riscv-vector-builtins-bases.h
 包含所有向量操作的namespace，声明外部的常量指针。

 #### riscv-vector-builtins-bases.cc
 声明 load store 的 enum 类型，定义了 unit stride, strided 和 indexed 三种。indexed 没有跟着 spec 写 indexed-unordered 和 indexed-ordered。
 定义了一个 vleff 和 vlsegff 的辅助函数去fold。调用 [gcc/gimple.h](https://github.com/gcc-mirror/gcc/blob/ae862e0e47cb2d62d7c624ab999a3bd8bd2914ef/gcc/gimple.h#L4) 里面的 gimple_call_num_args.   
 ```C++
 /* Return the number of arguments used by call statement GS.  */
 inline unsigned
gimple_call_num_args (const gcall *gs)
{
  return gimple_num_ops (gs) - 3;
}

inline unsigned
gimple_call_num_args (const gimple *gs)
{
  const gcall *gc = GIMPLE_CHECK2<const gcall *> (gs);
  return gimple_call_num_args (gc);
}
```
gcc/gimple.cc 里面的 `gimple_build_call_vec` 函数
```C++
/* Build a GIMPLE_CALL statement to function FN with the arguments
   specified in vector ARGS.  */

gcall *
gimple_build_call_vec (tree fn, const vec<tree> &args)
{
  unsigned i;
  unsigned nargs = args.length ();
  gcall *call = gimple_build_call_1 (fn, nargs);

  for (i = 0; i < nargs; i++)
    gimple_call_set_arg (call, i, args[i]);

  return call;
}
```
gcc/gimple.h 里的 `gimple_call_arg` 函数
```C++
/* Return the argument at position INDEX for call statement GS.  */

inline tree
gimple_call_arg (const gcall *gs, unsigned index)
{
  gcc_gimple_checking_assert (gimple_num_ops (gs) > index + 3);
  return gs->op[index + 3];
}
```
实现 vsetvl 和 vsetvlmax 

实现运算的操作看起来都是根据操作数类型转到对应的rtl.

 #### riscv-vector-builtins-functions.def   
 写 RVV intrinsic function 的名称，是否mask, 操作数符号和类型之类的。
 #### riscv-vector-builtins-shapes.cc   
 定义函数 shape NAME, 指向类`<NAME>_def`实例。   
 存在 rvv 0.7 的类定义，但是继承自 `misc_def` 结构体，`misc_def` 结构体继承自 `build_base` 结构体， `build_base` 结构体继承自 `function_shape` 类。   
 ##### function_shape 类   
 

 ```C++
 /* vget_def class.  */
struct vget_def : public misc_def
{
  bool check (function_checker &c) const override
  {
    poly_int64 outer_size = GET_MODE_SIZE (c.arg_mode (0));
    poly_int64 inner_size = GET_MODE_SIZE (c.ret_mode ());
    unsigned int nvecs = exact_div (outer_size, inner_size).to_constant ();
    return c.require_immediate (1, 0, nvecs - 1);
  }
};
```
 写intrinsic function的格式
 ```C++
 /* Declare the function shape NAME, pointing it to an instance
   of class <NAME>_def.  */
#define SHAPE(DEF, VAR) \
  static CONSTEXPR const DEF##_def VAR##_obj; \
  namespace shapes { const function_shape *const VAR = &VAR##_obj; }
  ```
 #### riscv-vector-builtins-shapes.h
 在 riscv_vector 的命名空间里声明 shapes 的命名空间
 #### riscv-vector-builtins-types.def
 定义数据类型的宏
 #### riscv-vector-builtins.cc
 看起来是在写intrinsic格式。
 #### riscv-vector-builtins.h
 riscv_vector 这个命名空间包括：   
 1.描述函数做什么的标识和读函数参数返回结果   
 2.定义用来识别RVV intrinsic需要的拓展的位值的宏   
 3.枚举 RVV 操作类型   
 4.
 声明 intrinsic 用到的数据类型，后缀的结构体。  
 function_base 类的定义   
 function_checker 类的定义   
 function_shape 类的定义   
 machine mode   
 *规定 intrinsic 特殊要求的bit表示*   
 **bit values 是哪里规定的呢？Full 'V' extension是什么呢？**    
 **`inline machine_mode` 是什么呢？之前看《编译原理》还有有个inline相关的参数，但是太久没继续看，已经忘得只剩下inline这个单词了。vector type 和 index type 跟 machine_mode 有什么关系呢？**   

 ```C++
 /* Bit values used to identify required extensions for RVV intrinsics.  */
#define RVV_REQUIRE_RV64BIT (1 << 0)	/* Require RV64.  */
#define RVV_REQUIRE_ELEN_64 (1 << 1)	/* Require TARGET_VECTOR_ELEN_64.  */
#define RVV_REQUIRE_ELEN_FP_32 (1 << 2) /* Require FP ELEN >= 32.  */
#define RVV_REQUIRE_ELEN_FP_64 (1 << 3) /* Require FP ELEN >= 64.  */
#define RVV_REQUIRE_FULL_V (1 << 4) /* Require Full 'V' extension.  */
#define RVV_REQUIRE_MIN_VLEN_64 (1 << 5)	/* Require TARGET_MIN_VLEN >= 64.  */
#define RVV_REQUIRE_ELEN_FP_16 (1 << 6) /* Require FP ELEN >= 32.  */
 ```
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

