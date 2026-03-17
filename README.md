# Lock-Free SPSC Queue (C++)

This project implements a **bounded lock-free single-producer single-consumer (SPSC) queue** in modern C++.  

The goal is to explore **low-latency concurrent data structures** and study the impact of:

- memory ordering
- cache-line contention
- synchronization primitives
- payload size
- queue capacity
- workload characteristics

To evaluate performance, the project includes a **custom benchmarking framework** that compares a lock-free queue against a mutex-based queue under various workloads.

---

## Features

### Lock-Free Queue Implementation
- Bounded **SPSC ring buffer**
- **Acquire / Release memory ordering**
- **Cache-line padded atomics** to prevent false sharing
- Non-blocking `try_push` / `try_pop` operations

### Benchmarking Framework
- Lock-free queue benchmark
- Mutex-based queue baseline
- Configurable experiment parameters:
  - queue capacity
  - operation counts
  - producer / consumer counts
  - payload sizes
- Warmup phase to stabilize caches and branch predictors
- Multi-threaded benchmarking

### Performance Metrics
The benchmark collects:

- Throughput (ops/sec)
- Latency statistics
  - average latency
  - p50
  - p95
  - p999
  - min / max

Each push and pop operation is timed using `std::chrono::steady_clock`.

---

## Queue Design

The queue uses a **bounded ring buffer** and two atomic indices:

- head → next element to be popped
- tail → next element to be pushed
    
Elements are stored in a fixed-size circular buffer.

Producer thread:
- write value → update tail (release)

Consumer thread:
- read value → update head (release)

Acquire loads ensure correct synchronization between producer and consumer **without locks**.

---

## False Sharing Prevention

The queue prevents cache-line contention by padding the atomic indices:

```cpp
struct alignas(64) PaddedAtomic {
    std::atomic<size_t> value;
    std::array<std::byte, 64 - sizeof(std::atomic<size_t>)> padding;
};
```
This ensures the producer and consumer operate on separate cache lines, preventing cache ping-pong.

---

## Benchmark
The benchmark automatically runs experiments across multiple configurations.

### Payload sizes tested

```markdown
16 bytes
64 bytes
256 bytes
```

### Example configuration parameters
- queue capacity
- push operations
- pop operations
- producer count
- consumer count

Example output:

```bash
====================================================================================================
Payload Size: 16
Capacity: 256
Producers: 1
Consumers: 1
Push Operations: 100000
Pop Operations: 100000
====================================================================================================

[Lock-Free SPSC Queue]
Throughput (ops/sec): 2.75e+07

PUSH:
  Avg latency (ns): 49
  p50 latency (ns): 0
  p95 latency (ns): 109
  p999 latency (ns): 109

POP:
  Avg latency (ns): 46
  p50 latency (ns): 0
  p95 latency (ns): 109
  p999 latency (ns): 109

[Mutex Queue]
Throughput (ops/sec): 6.9e+06
```

The lock-free queue typically achieves **4–5×** higher throughput than the mutex-based queue under contention.

---

## Build

```bash
mkdir build
cd build
cmake ..
make -j
```
---
## Run

```bash
./lf_queue_bench
```

The benchmark suite automatically runs all experiment configurations.

---

## Project Structure

```markdown
lock-free-spsc-queue
│
├── CMakeLists.txt
├── README.md
│
├── include
│   └── spsc_queue.hpp
│
├── benchmark
│   ├── benchmark_driver.hpp
│   ├── lockfree_queue_experiment.hpp
│   ├── mutex_queue_experiment.hpp
|   ├── payload.hpp
|   ├── stats.hpp
│   └── benchmark_report.txt
|  
└── src
    └── main.cpp
```
---

## Future Work

**Planned improvements:**
- CPU affinity pinning for more stable benchmarks
- exponential backoff strategies
- modulo vs bitmask indexing comparison
- multi-producer multi-consumer (MPMC) queue
- visualization of benchmark results
- integration with Google Benchmark
    
---

## Motivation

Lock-free data structures are fundamental to building high-performance concurrent systems, including:

- trading engines
- low-latency messaging systems
- task schedulers
- distributed runtime systems

This project explores how careful design of memory ordering, cache behavior, and synchronization primitives affects performance in modern multicore systems.
