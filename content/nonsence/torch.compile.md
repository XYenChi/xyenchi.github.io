---
title: "pytorch torch.compile"
date: "2026-02-01T00:00:03+00:00"
draft: false
---
[Dynamo 手册](https://docs.pytorch.org/docs/stable/user_guide/torch_compiler/torch.compiler_dynamo_overview.html)   

torch.compile 的前端 Dynamo 是：   
1.通用 Python 字节码解释器   
2.保留 Python 灵活性的同时允许图编译   

在函数编译过程中， Dynamo 解释 Python 字节码提取成 PyTorch 操作序列的 1 或多个 FX 图，后端可能会后期优化。   

function --Dynamo--> 1.FX graph 2.Python bytecode 3.guards   
TorchInductor 是支持将 Dynamo Graph 转换成 Triton 或者 C++/OpenMP 的其中一个后端。   

guard 会检查张量参数的形状。   
编译生成 guards 失败时就会重新编译。   

torch.compile 默认张量形状是固定的，并基于这一假设进行 guard。   
但也可以引入 dynamic shapes 接受不同形状的张量输入，避免每次形状不同都重新编译。   
