---
title: "RVV intrinsic的实现分析"
date: 2023-07-17T12:59:38+08:00
url: "/rvv_intrinsic_analysis/"
draft: false
---
### 包含的文件
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

#### riscv-v.cc
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
 规定 intrinsic 特殊要求的bit表示
 声明 intrinsic 用到的数据类型，后缀的结构体。
 function_base 类的定义
 function_checker 类的定义
 function_shape 类的定义
 #### riscv.md
 写rtl
 #### vector-iterators.md
 define_mode_iterator rtl   
 define_code_attr
 #### vector.md
