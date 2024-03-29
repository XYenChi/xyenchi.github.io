---
title: "RISC-V Vector 或 SIMD 类似拓展的应用举例子说明"
draft: false
---
### 线性代数部分   
#### 专有名词：   
[线性方程组](https://zh.wikipedia.org/zh-cn/%E7%BA%BF%E6%80%A7%E6%96%B9%E7%A8%8B%E7%BB%84)   
[齐次线性方程组](https://zh.wikipedia.org/zh-cn/%E7%BA%BF%E6%80%A7%E6%96%B9%E7%A8%8B%E7%BB%84#%E9%BD%90%E6%AC%A1%E7%BA%BF%E6%80%A7%E6%96%B9%E7%A8%8B%E7%BB%84)   
[增广矩阵](https://zh.wikipedia.org/wiki/%E5%A2%9E%E5%B9%BF%E7%9F%A9%E9%98%B5)
#### 还没有想好次级标题叫什么名字
不知道有没有必要从线性方程的求解开始讲起，简单带过一下好了。想象有很多未知数，如果没有限制条件，未知数的结果就想怎么编就怎么编，但是如果用方程组来当它们的限制条件，就可以缩小范围。用线性代数的方法求解就是，把未知数的位置对齐，将系数收集起来，称之为系数矩阵，对系数矩阵进行行变换化简，得到线性无关行向量个数，也就是矩阵的秩。秩小于未知数个数时，相当于限制条件没有太多，可以在一定范围内瞎编。而秩等于未知数个数时，想方便理解的话就还原一下原本的方程，如果是齐次线性方程，最后一行的未知数 x 常数之后还是等于0, 只能未知数自己也是0。以前学的教材上还有增广矩阵这个概念用来求等式右边不为0的情况，但是规律公式啥的都很难记，还是行变换之后想象一下原本的方程简单。
### 电路部分（线性齐次）
#### 专有名词：   
[基尔霍夫电流定律](https://zh.wikipedia.org/zh-hans/%E5%9F%BA%E7%88%BE%E9%9C%8D%E5%A4%AB%E9%9B%BB%E8%B7%AF%E5%AE%9A%E5%BE%8B#%E5%9F%BA%E7%88%BE%E9%9C%8D%E5%A4%AB%E9%9B%BB%E6%B5%81%E5%AE%9A%E5%BE%8B)
[基尔霍夫电压定律](https://zh.wikipedia.org/zh-hans/%E5%9F%BA%E7%88%BE%E9%9C%8D%E5%A4%AB%E9%9B%BB%E8%B7%AF%E5%AE%9A%E5%BE%8B#%E5%9F%BA%E7%88%BE%E9%9C%8D%E5%A4%AB%E9%9B%BB%E5%A3%93%E5%AE%9A%E5%BE%8B)
[节点分析](https://zh.wikipedia.org/zh-hans/%E7%AF%80%E9%BB%9E%E5%88%86%E6%9E%90)
#### 举个例子：   
本来是想拿个电路图举例子的，但是发现博客没调好，显示不了图片。等下一次心血来潮的时候修一修吧。   
大致过程就是，选一个参考节点，把所有节点标个号方便表示，给每一条支路都标个方向。找出受控源，列出支路方程，受控源正负极相连的方向如果跟你标的方向相同就写负号，相反就写正号。列出节点电压方程。改写成矩阵形式。就变成了一个方程组解的问题。


### 电路部分（线性非齐次）   
要写的也太多了。。。再说吧。
#### 专有名词：
#### 举个例子：   
### 关于射频仿真软件
之前在电子厂干天线的测试和量产前期的失效分析，因为看不到什么前途就在闲的时候看了一些天线仿真教学，但是也没太深入学习就跑路了，机缘巧合之下转行搞计算机。在开始学仿真的时候，学习到的就是不同的电尺寸选择的算法会不一样，还接触到了边界条件这些很有意思的概念。在学习RISCV相关的知识的时候，经常看到的例子就是计算RGB, 但是脑子里好像又有一些矩阵的其他用法，心血来潮，写下这篇博客，如果以后能学习到更多，那本文就是挖的坑，有空就来填，如果以后真的转行成功，在其他方向深入，那本文就是对过去的总结。