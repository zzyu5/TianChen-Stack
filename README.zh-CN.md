<p align="right">
  <a href="README.md">English</a> | <strong>简体中文</strong>
</p>

# TianChen Stack / 天辰软件栈

**TianChen Stack（天辰软件栈）** 是一个面向 **RISC-V 及其可扩展硬件扩展** 的大模型推理软件栈。当前重点关注 RISC-V / RVV / 自定义扩展等开放异构后端，后续将逐步扩展到 GPU、TPU、NPU 和其他 AI 加速器。

<p align="center">
  <img src="assets/images/Tianchen.png" alt="天辰软件栈概览" width="420">
</p>

这个仓库是 TianChen Stack 的总入口，用于组织三个相关项目：

| 层次 | 项目 | 状态 | 仓库 | 简介 |
|---|---|---:|---|---|
| 算子意图 / 前端层 | **IntentIR** | Done | [gitee.com/dongshan-community/IntentIR](https://gitee.com/dongshan-community/IntentIR) | 从 Triton、TileLang、CUDA 等优化 kernel 中提取可验证、可重调度的算子意图 |
| 编译器 / 后端层 | **TianChen Forge** | Doing | Coming soon | 面向新后端的模块化 AI compiler，通过结构化优化知识和硬件能力选择算法结构与映射方式 |
| LLM Serving / 运行时层 | **TianChen Serve** | Planned | Coming soon | 面向真实 LLM 推理服务的运行时层，后续用于连接 vLLM 式 serving 与异构后端 |

---

## Motivation

现代 LLM 推理系统通常依赖三类高度专业化的组件：

1. 面向特定硬件的高性能 kernel。
2. 面向特定 backend 的 compiler 规则和 schedule template。
3. 面向 GPU 集群的 LLM serving runtime。

这些方法在成熟 GPU 生态中非常有效，但在 RISC-V、自定义扩展、NPU、国产加速器和其他异构后端上会遇到新的问题：

- 高性能 kernel 中，算子语义和硬件 schedule 往往混在一起，难以直接复用到新后端。
- 直接让 LLM 生成或改写 kernel 仍然容易出错，尤其在小众硬件上缺少高质量训练语料时。
- compiler 的优化策略经常硬编码在 backend 或 template 中，新增硬件扩展时需要大量手工规则。
- 即使 kernel 和 compiler 能跑起来，最终仍需要接入真实 LLM serving 系统。

TianChen Stack 的目标是把这些隐含在实现中的知识显式化，形成从算子到编译器再到 serving 的可扩展软件栈。

---

## Project 1: IntentIR

**IntentIR** 是 TianChen Stack 的算子意图层。

它关注的问题是：

> 如何从已经优化过的 Triton、TileLang、CUDA 或 vendor kernel 中恢复可验证、可执行、可跨硬件重调度的算子意图？

高性能 AI kernel 通常把两类东西混在一起：

```text
what to compute      算子到底计算什么
how to schedule it   如何 tile、vectorize、thread-map、pipeline
```

这使得 kernel 很难直接审计、验证和移植。IntentIR 通过结构化 lifting 解决这个问题：

- 从多种 frontend kernel 中提取证据。
- 使用 LLM 作为受约束的 proposer，而不是 correctness authority。
- 通过 schema check、obligation check、source-execution agreement 和 certificate 进行验证。
- 将 kernel 表示为三层：
  - **Layer A:** Algorithmic Intent
  - **Layer B:** Portable Execution Structure
  - **Layer C:** Non-binding Schedule Hints

### Why IntentIR?

- 面向 LLM kernel 和复杂 AI 算子设计。
- 强调 correctness guarantee，而不是无约束生成。
- 让 LLM 处理结构化 lifting，而不是直接生成最终高性能 kernel。
- 分离语义和 schedule，因此可以保留性能线索，同时允许新后端重新调度。
- 不绑定某个专有 DSL，比单一 Triton/TileLang/CUDA 路线更适合作为多后端抽象层。
- 适合作为 RISC-V、RVV、NPU、自定义加速器等新后端的算子库基础。

Repository: [https://gitee.com/dongshan-community/IntentIR](https://gitee.com/dongshan-community/IntentIR)

---

## Project 2: TianChen Forge

**TianChen Forge** 是 TianChen Stack 的模块化编译器层。

IntentIR 解决“算子语义如何获得”的问题，但在新硬件上，仅有算子语义还不够。compiler 还需要决定：

- 用什么算法结构？
- 怎样做 partition / reduction / fusion？
- state 应该放在哪里？
- tile、vector width、parallel axis 如何选择？
- 不同 RISC-V extension 或 AI accelerator feature 下，哪些方案合法，哪些方案更优？

TianChen Forge 的目标是让 compiler 不再把这些优化知识全部硬编码在 backend 中，而是通过结构化的 **OptGuide** 资产来指导选择。

```text
Source / Provider / Runtime Evidence
        ↓
OptGuide Asset
        ↓
Hardware-conditioned Search / Selection
        ↓
Target Algorithm Structure
        ↓
Mapping / IR / Backend Realization
```

### Why TianChen Forge?

- 面向后端和 compiler 的模块化扩展。
- 适合 RISC-V 这类经常出现新 extension、新原子能力、新 vector/matrix 特性的硬件生态。
- 将算法结构、合法性约束和 mapping 偏好显式化。
- 减少 per-backend、per-kernel 的手写 patch。
- 可以结合硬件 profile 和 runtime evidence，自动搜索或选择更合适的后端实现。

Status: **Doing**  
Repository: **Coming soon**

---

## Project 3: TianChen Serve

**TianChen Serve** 是 TianChen Stack 的 LLM serving 层，当前处于规划阶段。

前两个项目解决的是算子与 compiler 问题，而最终目标是让新后端真正进入真实 LLM 推理服务。TianChen Serve 将面向 vLLM 式 online serving，关注 prefill、decode、KV cache、batching、routing 和异构 engine 接入等问题。

当前阶段我们只将它作为软件栈的第三层入口：

```text
IntentIR gives operator intent.
TianChen Forge builds backend/compiler support.
TianChen Serve connects the result to real LLM inference.
```

Status: **Planned**  
Repository: **Coming soon**

---

## TianChen Stack at a Glance

```text
TianChen Stack
├── IntentIR
│   └── Certified semantic lifting for optimized AI kernels
│
├── TianChen Forge
│   └── OptGuide-driven modular compiler for extensible backends
│
└── TianChen Serve
    └── Runtime layer for real LLM serving on heterogeneous backends
```

---

## Roadmap

- [x] IntentIR: certified semantic lifting for optimized AI kernels
- [ ] TianChen Forge: modular compiler layer for RISC-V and extensible backends
- [ ] OptGuide assets for algorithm and mapping guidance
- [ ] TianChen Serve: LLM serving integration layer
- [ ] RISC-V / RVV backend case studies
- [ ] Heterogeneous accelerator support
- [ ] End-to-end LLM inference demo

---

## License

TBD

---

## Contact

This repository is under active development.

For IntentIR, see:

[https://gitee.com/dongshan-community/IntentIR](https://gitee.com/dongshan-community/IntentIR)
