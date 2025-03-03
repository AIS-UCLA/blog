---
title: "CPU vs GPU Memory Models"
author: Gabriel Castro
---

# JMM and PTX Memory Model

Recently in cs131, Eggert was lecturing about the Java Memory Model (JMM) which defines how threads behave when accessing shared memory space. This was in the context of CPU multi-threading applications. So it got me thinking, what about for GPUs? How would their memory models differ? With their higher focus on parallel processing and much much greater thread count, how are these numerous threads expected to behave when accessing the same memory space? If you’ve programmed in CUDA before you’ll realize how important this type of communication and synchronization between threads are when accessing the same memory addresses. In this short post, I’ll explain how NVIDIA GPUs in particular implements the PTX memory consistency model in their architectures.

## weak memory model
Like most CPU memory models the PTX memory model is weak, in the sense that the compiler can do whatever optimizations it wants to the source code and can produce assembly instructions that reorder specific instructions for performance reasons (strong models would enforce a specific ordering for the instructions to be executed in). In a weak memory model, the programmer can use synchronization operations to enforce causality, so that threads will wait for other threads to load the data, if they have to use it i.e. in a producer-consumer scenario.

The model also defines the visibility of the shared memory space i.e. how different threads see what changes happen to the memory and whether those changes are even visible at all to them. So the memory model implements coherence so the threads can have a consistent view of how values change at a single address. Other guarantees specified by the model include atomicity and preventing “out-of-thin-air” values (registers can’t load values from memory that weren’t there in the first place). These kinds of rules are quite normal for what the programmer would expect when writing multithreading apps. 


## scopes
So now what makes GPU memory models a bit different (NVIDIA in particular)? Well they introduce the concept of scopes to their memory models.

Recall, there are a lot more threads spawned in a GPU and CUDA organizes them into a thread hierarchy for efficiency/synchronization purposes, as such NVIDIA also adds scopes to their PTX memory model. Similar to how threads are grouped together in warps, blocks, etc in the CUDA, you can think of scopes as a set of threads.

For instance, a PTX memory instruction can specify multiple scopes: `ld{.scope} =  {.cta, .cluster, .gpu, .sys};`

And so when you have 2 different memory instructions that include each other in their scope i.e. `st.release.gpu, ld.acquire.gpu`, they’re considered **morally strong** to one another. However, if 2 different memory instructions don’t include each other in their scope like `st.release.cta, ld.acquire.cta` (since threads can belong to different CTAs) they’re **not morally strong**.

## weak + scopes = profit
Putting it all together, the guarantees from the weak memory model (coherence, causality, etc.) only apply between **morally strong** memory instructions. This means that memory instructions operating in the same scope as one another will adhere to the same rules of the weak memory model. While instructions in differing scopes are not guaranteed to follow the rules of the weak memory model, so a thread in some cta may not see the same changes in memory made from another thread in a different cta leading to inconsistent and less deterministic behavior. 

This scope-based approach allows for more flexibility and optimization, as now there is more synchronization control among different scopes and reduced overhead for thread communication in smaller scopes. Additionally data races are allowed in the PTX memory model, which is different from other GPU memory models like OpenCL, which requires data-race freedom in their written programs. (so NVIDIA leaves it up to the programmer with dealing with race conditions, *good luck*)

##### this post was inspired by one of Eggert’s lectures :P

### references:
[PTX ISA Docs Ch.8](https://docs.nvidia.com/cuda/parallel-thread-execution/#scope-and-applicability-of-the-model)\
[A Formal Analysis of the NVIDIA PTX Memory Consistency Model](https://dl.acm.org/doi/pdf/10.1145/3297858.3304043)
