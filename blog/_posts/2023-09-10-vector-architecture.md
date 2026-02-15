---
layout: post
title:  "Vector architecture 101"
date:   2023-09-10
tags: ["computer architecture" ,"hardware", "SIMD", "SIMT", "vector architecture", "gpu"]
excerpt: "Place excerpt here"
hide: true
---

[GPUs are all the hype at the moment](https://stratechery.com/2023/nvidia-on-the-mountaintop/) and if you don't have access to a 1000 GPU cluster, you are considered [GPU-poor](https://www.semianalysis.com/p/google-gemini-eats-the-world-gemini). The surge is driven by a wide reception of [Large](https://openai.com/gpt-4) [Language](https://blog.google/technology/ai/google-palm-2-ai-large-language-model/) [Models](https://ai.meta.com/llama/) (LLMs). However, GPUs have existed for a while now and one thing that sits at the heart of GPUs is vector computation.  

# Vector architecture components

![Vector architecture](/images/vector-arch/vector_arch_overview.png)

Vector architectures work well for data-parallel applications. They mainly consist of the following:
1. **Vector memory unit**: To load/store "grouped" vector data from memory at higher bandwidth than scalar accesses. Typically, they support contiguous and strided memory accesses. A dedicated vector engine, will also support gather/scatter operations to allow arbitrary addresses.

2. **Vector registers**: Vector registers provide the same benefits as scalar registers but limit the supported hardware vector length. The earliest vector architectures were [memory](https://dl.acm.org/doi/abs/10.1145/1479992.1480022)-to-[memory](https://inis.iaea.org/collection/NCLCollectionStore/_Public/10/462/10462027.pdf) vector machines allowing them to support arbitrary vector lengths but had a high startup cost and only really worked for long vectors > 100 elements.  

3. **Scalar registers**, to allow for scalar-vector computation in addition to scalar support.


4. **Vector functional units**: These units often support computing multiple vector elements in the same cycle using parallel lanes.
![Vector Functional Units](/images/vector-arch/vec-func-unit.png)

    * VFU: Vector functional units - multiply, add, shift, logical

    * Lanes, pipes: Parallel execution units of the same VFU.

    * Chiming: Unit time taken to process a vector register, of length M on the N lanes = `M/ N` . This is similar to loop strip-mining.



# Kinds of vector machines

1. Traditional long vector machines: Long vector machines [6,7,8] have a scalable/long "vector length" and use chiming internally. [6,7] don't even have a vector registers, they are memory to memory. They just copy a long vector typically > 100 elements from memory to execution units in chimes, and execute CISC-like instructions.
2. Packed-SIMD: They have a 1:1 mapping of vector register to the vector lane. 

# Tradeoffs in vector architecture design

Number of lanes, Coupling with scalar compute, Memory bandwidth etc 

# Vector machine properties

 - What is hardware vector length? It is defined by the vector register length.
 - How are vector operations pipelined with dependencies? Chaining
 - Supporting conditional branch - Predication
 - Memory access patterns
 - Packed SIMD vs decoupled engine?

# History

TI-ASC and STAR-100 were memory to memory vector machines. They did not have a hardware limitation on the vector register length and could potentially deal with very long vector lengths. They had CISC-like instructions which could do matrix multiplication in one instruction. They were however, slow for scalar computation and needed long vector lengths of over 100 to show efficacy. They introduced masking vector lanes and gather/scatter memory operations.

Cray-1 integrated vector registers and VFUs together with scalar computation. Vector component was more of an "add-on" to the scalar machine. Memory bandwidth was limited to 1 load or store per cycle! Still large vector machines with 64 word elements per vector register. They also introduced chaining to avoid expensive vector register read/write where they forward the result of instruction to the next as operand. This allowed efficient usage of parallel VFUs and increased FLOPs/cycle. Remember, chiming can take a long time to finish a single vector operation along a long vector length so chaining is really effective instead of waiting for the chime.

1990s was the "killer micros" time when cmos scaling proved that multiprocessing was faster than vectors. 

There was 2 directions at this point, since scalar computation was scaling so fast: 1. Short-vectors coupled with scalar cores - packed-SIMD 2. vector engines and processors, which would benefit "SIMD" parallel workloads with reduced control and memory overhead.

Tarantula: Attaches a 16-lane vector engine to a scalar core and bypasses L1 cache to connect to L2 for higher bandwidth. 

Cray Black Widow: Vector processor

Packed SIMD

Packed-SIMD (also called short-vector SIMD) tightly couples vector lanes with the scalar core. Each vector register maps directly to lanes that execute in parallel on the same instruction stream — this is the ISA-level model used by x86 SSE/AVX and ARM NEON. Packed-SIMD is a good fit when:

- The data-parallel work fits in short vectors (e.g. 4, 8, 16 elements).
- The control flow is uniform or can be predicated/packed efficiently.

Advantages of packed SIMD:

- Low control overhead because lanes follow the scalar core's instruction stream.
- Very efficient for compute-bound inner loops (dense linear algebra, multimedia).

Limitations:

- Fixed, relatively small vector width limits the per-instruction parallelism.
- Handling irregular memory patterns is harder — gather/scatter support is recent.

SIMT and GPUs

SIMT (Single Instruction Multiple Threads) is the execution model most GPUs use — logically similar to SIMD but implemented as many lightweight threads grouped into warps/wavefronts. Differences from classic SIMD:

- SIMT exposes per-thread control (threads can diverge), but divergence is typically serialized within a warp.
- GPUs provide very wide parallelism (thousands of threads) with many vector lanes and deep multithreading to hide memory latency.

GPUs also couple wide vectorization with explicit memory hierarchy and high memory bandwidth, making them ideal for massively data-parallel workloads where throughput is more important than single-thread latency.

Vectorization in compilers and runtimes

Modern compilers try to auto-vectorize loops where possible: they perform dependence analysis, alignment checks, and loop transformations (unrolling, strip-mining) to convert scalar loops into vector instructions. When auto-vectorization is not possible, developers use intrinsics or libraries to express packed-SIMD operations directly.

Practical considerations

- Memory bandwidth and locality: Vector units are starved without sufficient memory bandwidth. Streamlined memory accesses (contiguous/strided) and prefetching improve utilization.
- Alignment: Aligned loads/stores are faster; when alignment can't be guaranteed, compilers emit peel/trailer code or use masked loads.
- Masking & predication: Support for selective lane execution avoids expensive branches and is essential for irregular control.
- Gather/Scatter: Useful for indirect/irregular accesses, but often slower than contiguous loads — design algorithms to minimize their use or hide latency.
- Chaining and latency hiding: Chaining (forwarding intermediate results) and deep multithreading (GPUs) help keep functional units busy despite long vector lengths.

When to use vectorization

- Use vectorization for data-parallel kernels (linear algebra, image processing, signal processing, many ML primitives).
- Prefer packed-SIMD for short, highly-optimized kernels on CPUs; prefer wide SIMT-style parallelism on GPUs for throughput-oriented tasks.

Closing thoughts

Vector architectures are a powerful abstraction for expressing data parallelism. Over time the design space has shifted from very long, memory-to-memory vector machines to flexible packed-SIMD and wide-SIMT models that balance control, memory, and compute. Understanding the tradeoffs — vector length, coupling with scalar cores, memory bandwidth, and control mechanisms like masking — helps you pick the right platform and algorithm strategy.

Further reading

- Patterson & Hennessy, Computer Architecture: A Quantitative Approach (for historic machines like Cray)
- Research papers on TI-ASC and STAR-100 (early memory-to-memory designs)
- Documentation and ISA guides for x86 AVX/AVX-512 and ARM SVE/NEON for packed-SIMD details

---




