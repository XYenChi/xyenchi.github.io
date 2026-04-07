---
title: "PyTorch for RISC-V 的现状和发展"
url: "/pytorch-riscv/"
draft: false
---
随着人工智能技术的爆发式增长，深度学习框架与底层硬件的结合变得前所未有的紧密。在软件端，PyTorch 凭借其易用性和动态图机制，已经成为学术界和工业界最受欢迎的深度学习框架之一；而在硬件端，RISC-V 架构凭借其开源、精简、模块化的特性，正在从物联网（IoT）领域迅速向高性能计算和人工智能加速领域渗透。

“当最火的开源 AI 框架遇上最具潜力的开源指令集”，PyTorch on RISC-V 的生态构建不仅是打破传统 x86 和 ARM 架构垄断的关键一步，更是推动 AI 算力普惠化和底层硬件创新的重要力量。本文将深入探讨 PyTorch 在 RISC-V 架构上的发展现状、面临的挑战以及未来的演进方向。   

### 为什么我们需要 PyTorch on RISC-V？
在讨论现状之前，首先需要理解这一组合的内在逻辑：   

##### 打破算力垄断与定制化需求：
传统的 CPU/GPU 架构面临高昂的授权费用和技术壁垒。RISC-V 的开源特性允许开发者根据特定 AI 场景（如边缘推理、自动驾驶控制）定制扩展指令。PyTorch 的原生支持能够让这些定制化芯片迅速无缝接入成熟的 AI 算法生态。   

#### RISC-V 向量扩展（RVV）的潜力：
深度学习的核心是海量的张量（Tensor）运算。RISC-V 推出的向量扩展标准（RVV, RISC-V Vector Extension）天生契合矩阵乘法和并行计算。通过 PyTorch 调动 RVV，可以极大提升 RISC-V 芯片的 AI 推理与训练性能。   

#### 边缘 AI（Edge AI）的崛起：
未来的 AI 不仅存在于云端数据中心，更将广泛部署于智能终端。RISC-V 功耗低、面积小的优势使其成为边缘 AI 的绝佳载体，而 PyTorch（特别是面向边缘设备的 ExecuTorch）则是连接算法与端侧硬件的桥梁。   

### PyTorch for RISC-V 的发展现状
从当前实际情况来看，PyTorch 在 RISC-V 上尚未形成官方完整支持，整体处于“可运行但不可高效使用”的阶段。首先，在基础可用性方面，PyTorch 已经可以通过源码编译在 RISC-V Linux 系统上运行，但通常需要关闭 CUDA、QNNPACK、FBGEMM 等关键优化组件，仅支持 CPU 推理路径 。这意味着其性能与 x86 或 ARM 平台相比存在明显差距。此外，构建过程复杂，对工具链、依赖版本要求严格，生态仍不完善。其次，在上游支持层面，目前 PyTorch 官方仓库中仍缺乏完善的 RISC-V CI 测试、预编译二进制以及系统级适配。社区虽有零散贡献（如 build 脚本、Docker 环境等），但整体尚未进入主线支持范畴   

#### 基础移植与运行环境的打通
早期的 RISC-V 开发者如果想运行 PyTorch，需要经历痛苦的源码交叉编译过程，并手动解决大量底层依赖（如缺少对 RISC-V 支持的数学库）。如今，得益于 RISC-V 国际基金会、中科院软件所、玄铁、SiFive、RISE 基金会等机构的持续贡献，PyTorch 及其核心依赖（如 NumPy, SciPy）在 RISC-V 平台上的编译已经大幅简化。在 Milk-V Pioneer、LicheePi 4A 以及 VisionFive 2 等 RISC-V 开发板上，开发者已经可以通过预编译的 Wheels 包或相对简单的步骤原生运行 PyTorch。   

#### 底层数学库的优化
PyTorch 的性能高度依赖于底层的 BLAS（Basic Linear Algebra Subprograms）库。目前，[OpenBLAS](https://github.com/OpenMathLib/OpenBLAS?tab=readme-ov-file#risc-v) 和 Eigen 等核心数学计算库已经初步合入了对 RISC-V RVV 1.0 标准的支持。这意味着在 RISC-V 开发板上运行 PyTorch 张量运算时，系统已经能够调用向量指令集进行硬件加速，而非仅仅依赖低效的标量计算。

#### 框架上游生态的接纳
在开源社区的推动下，PyTorch 上游代码库正在逐渐接纳 RISC-V 相关的 Patch。CI/CD（持续集成/持续交付）管道中开始引入基于 RISC-V 模拟器或真实硬件的测试节点，   

### 当前面临的挑战
尽管进展显著，但 PyTorch for RISC-V 距离在生产环境中大规模应用仍有一段距离：   

#### PyTorch 2.0 编译栈的缺位： 
PyTorch 2.0 引入了极其重要的 torch.compile 特性，依赖于 TorchInductor 和 OpenAI Triton 等技术来自动生成优化的底层代码。然而，目前这些编译器后端对 RISC-V 的支持仍然非常薄弱，导致 RISC-V 芯片难以享受 PyTorch 2.0 带来的巨大性能红利。   

#### 与成熟架构的性能鸿沟： 
相比于 x86 经过几十年的指令集优化（如 AVX-512）以及 ARM 完善的生态（如 Neon/SVE），RISC-V 的算子手工调优才刚刚起步。面对动辄数百 GB 甚至 TB 级别大模型的微调或推理，单靠 CPU 的 RVV 依然捉襟见肘。   

### 未来的发展方向
展望未来，PyTorch for RISC-V 的演进将主要集中在以下几个核心方向：   

##### 争取 Tier-1 级别的官方支持
社区的首要目标是推动 RISC-V 成为 PyTorch 官方的 Tier-1（或至少稳固的 Tier-2）支持架构。这意味着 PyTorch 官方将提供针对 RISC-V 的 pre-built binaries（预编译包），开发者可以通过简单的 `pip install torch` 直接在 RISC-V 设备上安装体验，彻底扫除使用门槛。   

#### 实现算子级优化（uKernel）
核心任务是构建类似 ARM KleidiAI/XNNPACK 的优化库，实现基于 RVV 的 SIMD kernel，优化关键算子，提升 ATen 层性能。   

#### Triton/TileLang RISC-V 后端支持
目前，Triton 和 TileLang 对 RISC-V 的原生、开箱即用支持均处于早期或实验性探索阶段。Triton 官方主线分支高度绑定 GPU 架构。RISC-V 并不是官方定义的 Tier-1 支持目标。TileLang 作为一个较新的、采用 Pythonic 语法的轻量级高性能 DSL，其核心策略是利用 TVM 作为底层的编译基础设施，从而生成 C/C++ 或特定硬件的底层代码。

#### 深度拥抱 PyTorch 2.0 编译器技术
未来优化的重点将从“手写汇编算子”转向“编译器自动生成”。通过完善 LLVM 的 RISC-V 后端，并将 RISC-V 架构深度集成到 TorchInductor 和 MLIR 等编译栈中，PyTorch 能够根据具体的 RISC-V 芯片特性（如向量寄存器长度 VLEN）动态生成最优的执行代码。这不仅能提升性能，还能有效解决生态碎片化问题。   

#### 异构计算：RISC-V CPU + NPU/GPU 协同
未来的 AI 芯片往往是异构的。RISC-V 将更多地扮演高性能主控协处理器的角色，而密集的矩阵计算则交给专门的 NPU（神经网络处理器）或基于开放标准的 GPU。因此，PyTorch 在 RISC-V 上的发展方向将包含如何高效地调度外部加速器。例如，基于 OpenXLA 等技术，建立从 PyTorch 经过 RISC-V 控制面，最终在定制 NPU 上执行计算的高效数据通路。   

#### 聚焦大模型与端侧推理部署
随着大语言模型（LLM）向端侧下沉，PyTorch 生态下的 ExecuTorch 和基于 RISC-V 的结合将大放异彩。未来的发展将致力于在资源受限的 RISC-V 终端芯片上，通过量化（INT8/INT4）、剪枝等技术，实现 PyTorch 模型的超低功耗推理，赋能智能汽车、机器人和下一代可穿戴设备。   

### 结语
PyTorch for RISC-V 的融合，本质上是软件开源与硬件开源的顶峰相见。虽然目前它仍处于从“能用”向“好用”过渡的攻坚阶段，但随着全球芯片产业链对自主可控和灵活定制的呼声日益高涨，以及全球开源开发者的共同努力，RISC-V 必将成为承载 PyTorch AI 算力的一片广阔新大陆。未来的软硬件协同设计，势必将围绕这一极具活力的双开源生态展开。   