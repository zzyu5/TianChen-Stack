<p align="right">
  <strong>English</strong> | <a href="README.zh-CN.md">简体中文</a>
</p>

# TianChen Stack

**TianChen Stack** is an LLM inference software stack for **RISC-V and extensible hardware extensions**. The current focus is on RISC-V / RVV / custom extensions and open heterogeneous backends. Future versions will gradually expand toward GPUs, TPUs, NPUs, and other AI accelerators.

<p align="center">
  <img src="assets/images/Tianchen.png" alt="TianChen Stack overview" width="420">
</p>

This repository is the entry point for the TianChen Stack and organizes three related projects:

| Layer | Project | Status | Repository | Description |
|---|---|---:|---|---|
| Operator intent / frontend layer | **IntentIR** | Done | [github.com/zzyu5/IntentIR](https://github.com/zzyu5/IntentIR) | Extracts verified and retargetable operator intent from optimized Triton, TileLang, CUDA, and vendor kernels |
| Compiler / backend layer | **TianChen Forge** | Doing | Coming soon | A modular AI compiler layer for new backends, using structured optimization knowledge and hardware capabilities to select algorithms and mappings |
| LLM serving / runtime layer | **TianChen Serve** | Planned | Coming soon | A runtime layer for real LLM inference, connecting vLLM-style serving with heterogeneous backends |

---

## Motivation

Modern LLM inference systems usually depend on three kinds of specialized components:

1. Hardware-specific high-performance kernels.
2. Backend-specific compiler rules and schedule templates.
3. LLM serving runtimes optimized for GPU clusters.

These methods work well in mature GPU ecosystems, but they become fragile when targeting RISC-V, custom extensions, NPUs, domestic accelerators, and other heterogeneous backends.

The main challenges are:

- Optimized kernels often entangle operator semantics with hardware-specific schedules.
- Direct LLM-based kernel generation or kernel-to-kernel translation is still error-prone, especially for niche hardware with limited high-quality kernel corpora.
- Compiler optimization strategies are often hard-coded in backend rules or templates, making new hardware extensions expensive to support.
- Even when kernels and compilers are available, the resulting backend still needs to be integrated into real LLM serving systems.

TianChen Stack aims to make this hidden knowledge explicit, creating an extensible path from operators to compilers to LLM serving.

---

## Project 1: IntentIR

**IntentIR** is the operator-intent layer of TianChen Stack.

It addresses the following question:

> How can we recover verified, executable, and retargetable operator intent from optimized Triton, TileLang, CUDA, or vendor kernels?

High-performance AI kernels often mix two kinds of information:

```text
what to compute      the operator semantics
how to schedule it   tiling, vectorization, thread mapping, pipelining
```

This makes kernels difficult to audit, validate, and retarget. IntentIR solves this through structured semantic lifting:

- It extracts evidence from multiple frontend kernel sources.
- It uses LLMs as constrained proposers, not as correctness authorities.
- It validates recovered artifacts through schema checks, obligation checks, source-execution agreement, and certificates.
- It represents kernels in three layers:
  - **Layer A:** Algorithmic Intent
  - **Layer B:** Portable Execution Structure
  - **Layer C:** Non-binding Schedule Hints

### Why IntentIR?

- Designed for LLM kernels and complex AI operators.
- Correctness-first instead of unconstrained generation.
- Lets LLMs solve structured lifting tasks rather than directly generating final high-performance kernels.
- Separates semantics from schedules, preserving performance hints while allowing new backends to retune.
- More backend-neutral than a single frontend-specific DSL such as Triton, TileLang, or CUDA.
- Useful as a foundation for operator libraries on RISC-V, RVV, NPUs, and custom accelerators.

Repository: [https://github.com/zzyu5/IntentIR](https://github.com/zzyu5/IntentIR)

---

## Project 2: TianChen Forge

**TianChen Forge** is the modular compiler layer of TianChen Stack.

IntentIR provides operator intent, but intent alone is not enough. On new hardware, a compiler still needs to decide:

- Which algorithm structure should be used?
- How should partitioning, reduction, and fusion be organized?
- Where should state be materialized?
- How should tile shapes, vector widths, and parallel axes be selected?
- Under different RISC-V extensions or accelerator features, which choices are legal and which choices are preferred?

TianChen Forge aims to prevent such optimization knowledge from being hard-coded entirely inside backend templates. Instead, it uses structured **OptGuide** assets to guide compiler decisions.

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

- Provides a modular compiler layer for backend extension.
- Fits RISC-V-like ecosystems where new extensions, new primitive capabilities, and new vector/matrix features may appear frequently.
- Makes algorithm structures, legality constraints, and mapping preferences explicit.
- Reduces per-backend and per-kernel manual patches.
- Uses hardware profiles and runtime evidence to search or select better backend implementations.

Status: **Doing**  
Repository: **Coming soon**

---

## Project 3: TianChen Serve

**TianChen Serve** is the LLM serving layer of TianChen Stack and is currently planned.

The first two projects address the operator and compiler layers. The final goal is to make new backends usable in real LLM inference services. TianChen Serve will target vLLM-style online serving and will focus on prefill, decode, KV cache, batching, routing, and heterogeneous engine integration.

At this stage, we keep it as the third layer of the stack:

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

[https://github.com/zzyu5/IntentIR](https://github.com/zzyu5/IntentIR)
