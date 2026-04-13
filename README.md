# Safe Flow Decomposition

A high-performance C++20 application layer engineered for identifying maximal safe paths in complex directed flow networks. This tool leverages a custom **AVX-512 vectorized graph engine** to solve flow decomposition problems with microsecond-level latency, achieving a **3-4x empirical speedup** over traditional scalar implementations.

## The Core Application

In quantitative network analysis, decomposing a flow network into a set of safe, disjoint paths is a common bottleneck. This project optimizes the "extraction" phase by offloading path-finding and graph traversal to a hardware-accelerated backend.

- **Maximal Path Extraction**: Systematically identifies disjoint safe paths by evaluating per-edge flow constraints.
- **Bottleneck Analysis**: Uses a vectorized SPFA (Shortest Path Faster Algorithm) to rapidly locate bottleneck edges and saturate them in parallel.
- **Zero-Copy Residual Tracking**: Maintains residual capacity states using bitmask-driven updates, preserving the cache-aligned integrity of the underlying hardware-optimized graph.

## Performance Architecture

To achieve extreme throughput, this application is built on a specialized infrastructure designed for modern x86_64 CPUs:

- **AVX-512 Vectorization**: Processes 16 edge relaxations in a single CPU clock cycle using `_mm512_i32gather_epi32` and masked scatter intrinsics.
- **Lock-Free Synchronization**: Utilizes a Michael-Scott lock-free queue and specialized atomic memory ordering (`memory_order_acquire/release`) to eliminate thread contention during concurrent path exploration.
- **64-Byte Cache Alignment**: Employs a **Compressed Sparse Row (CSR)** format with custom `AlignedAllocator<64>` to eliminate cache-line splitting and maximize prefetcher efficiency.

---

## 1. Build the application and benchmarks
mkdir build && cd build
cmake .. -DCMAKE_BUILD_TYPE=Release
make -j$(nproc)

## 2. Run the Safe Flow Benchmark
### Validates the 3-4x speedup claim against the scalar baseline
./benchmarks/benchmark_safe_flow

## 3. Run Correctness Tests
### Ensures flow conservation and path independence
./tests/graph_tests --gtest_filter=SafeFlowTest.*
