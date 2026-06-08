# Lab 6 - Caches Lab Report
**Name:** [Student Name]
**Email:** [Student Email]

## Configuration Analysis
In this lab, I examined various cache configurations across different executables to evaluate the tradeoff between performance (miss rate) and hardware cost.

### Chosen Configurations & Justification
From the generated statistics, we observed that increasing both the cache size and the associativity generally leads to lower miss rates. For example, in the `matrix-mul-row-major` executable with an LRU replacement policy, the miss rate drops significantly as cache size increases from 1024 bytes up to 16384 bytes. 

Increasing associativity from 1 (Direct Mapped) to 2, 4, and 8 ways also shows a diminishing return in miss rate reduction. A 4-way set associative cache often presents the best balance:
* **Performance:** It significantly reduces conflict misses compared to a direct-mapped cache, often performing very close to an 8-way set associative cache.
* **Cost:** Hardware cost (comparators, multiplexers, and routing logic) scales linearly with associativity. An 8-way cache requires double the parallel tag checks and routing complexity of a 4-way cache, often making the slight improvement in miss rate not worth the extra area, power, and access time overhead. 

Therefore, a **4096-byte or 8192-byte 4-way set-associative cache with LRU replacement** provides an excellent "sweet spot" for general workloads, minimizing the penalty of frequent cache misses without overcommitting die space to logic gates.

## Performance Chart

Below is the plot tracking the miss rate across different cache sizes and associativity levels for the `matrix-mul-row-major` unified cache using the LRU replacement policy.

![Miss Rate vs Cache Size](assets\hello_c.png)

## Observations Across Executables
When observing the 4 distinct executables (`hello_c`, `hello_cpp`, `matrix-mul-row-major`, `matrix-mul-col-major`), several themes emerge:

1.  **Spatial Locality Matters:** The most striking difference is between `matrix-mul-row-major` and `matrix-mul-col-major`. In C/C++, matrices are stored in row-major order. Consequently, the row-major multiplication accesses memory sequentially, taking full advantage of cache blocks (high spatial locality). The column-major multiplication strides through memory, heavily polluting the cache and resulting in a much higher miss rate, even at larger cache sizes.
2.  **Instruction Cache vs Data Cache:** For the "Hello World" programs (`hello_c` and `hello_cpp`), the instruction cache accounts for a significant portion of memory accesses compared to the data cache. While both program types are simple, C++ often has a larger instruction footprint due to the standard library streams and runtime setup, resulting in longer initial memory traces and slightly varied cache warm-up behavior.
3.  **Diminishing Returns on LRU vs FIFO:** Across most executables, LRU slightly outperforms FIFO, especially in higher associativity caches. However, the margin is often small, meaning for highly cost-constrained embedded systems, FIFO might be a reasonable alternative to save state bits.

Overall, the common theme is that **algorithm design (software) heavily dictates cache performance**. Even an optimally configured 16KB 8-way cache cannot fully compensate for the poor spatial locality of the column-major traversal.
