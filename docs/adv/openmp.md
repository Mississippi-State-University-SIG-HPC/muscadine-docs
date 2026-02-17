# Getting Started with OpenMP
Resources:  [OpenMP Official Site](https://www.openmp.org)

**OpenMP** is a widely used API for shared-memory parallel programming, this guide shows how to leverage it effectively on HPC systems such as Muscadine. We'll cover the basics, practical examples, and optimization strategies.

---

## What is OpenMP?

**OpenMP (Open Multi-Processing)** is a standard API for parallel programming in shared-memory environments. It allows you to parallelize loops, sections, and tasks using compiler directives (`#pragma` in C/C++).

**Key features:**

- **Incremental parallelization:** Parallelize sections of existing serial code.
- **Portable:** Supported on most HPC clusters.
- **Scalable:** Suitable for multi-core nodes on HPC systems.

```{note}
OpenMP does **NOT** scale across nodes. OpenMP is useful when optimizing existing code or when used in conjunction with [MPI](/sub/mpi-basics.md).
```

--- 

## Setting Up Your Environment

Before writing OpenMP programs, ensure your a compatible compiler is loaded:

```bash
module load gcc
# or
module load aocc
```

Check OpenMP support:

```bash
gcc -fopenmp -o test_openmp test_openmp.c
```

---

## Your First OpenMP Program

Parallelizing a simple `for` loop in C:

```c
#include <stdio.h>
#include <omp.h>

int main() {
    int i;
    int n = 16;

    #pragma omp parallel for
    for (i = 0; i < n; i++) {
        printf("Thread %d processing iteration %d\n", omp_get_thread_num(), i);
    }

    return 0;
}
```

**Compile and run:**

```bash
gcc -fopenmp -O2 -o hello_omp hello_omp.c
export OMP_NUM_THREADS=4
./hello_omp
```

**Output:** Shows iterations distributed across threads.

---

## OpenMP Directives

### Parallel Loops

```c
#pragma omp parallel for schedule(static, 4)
for (int i = 0; i < N; i++) {
    array[i] = compute(i);
}
```

- `schedule(static, 4)` splits work into chunks of 4 iterations per thread.
- `schedule(dynamic)` useful when iteration times vary.

### Parallel Sections

```c
#pragma omp parallel sections
{
    #pragma omp section
    compute_part1();

    #pragma omp section
    compute_part2();
}
```

### Reduction

```c
double sum = 0.0;

#pragma omp parallel for reduction(+:sum)
for (int i = 0; i < N; i++) {
    sum += array[i];
}
```

**Reduction** avoids race conditions in summing or accumulating values.

---

## Best Practices for HPC

1. **Set `OMP_NUM_THREADS=$SLURM_CPUS_PER_TASK`:** Use the `--cpus-per-task` slurm directive to control threading
2. **Avoid false sharing:** Align data to prevent multiple threads from writing to the same cache line.
3. **Use `schedule(dynamic)` for irregular workloads:** Minimizes load imbalance.
4. **Profile before tuning:** Use tools like `gprof`, `perf`, or HPC-specific profilers.
5. **Hybrid MPI + OpenMP:** Combine MPI across nodes with OpenMP within (numa) nodes for optimal scaling.

---

## Advanced HPC Tips

* **Thread affinity:** Bind threads to cores to reduce cache thrashing:

  ```bash
  export OMP_PROC_BIND=close
  export OMP_PLACES=cores
  ```
* **Nested parallelism:** Enable with `omp_set_nested(1)` if your code spawns parallel regions inside others.
* **Memory considerations:** Ensure your data structures fit within NUMA domains to maximize memory bandwidth.

---

```{tip}
Start small: parallelize the most computationally heavy loops first, and gradually extend OpenMP pragmas across your HPC code. Measure scaling at each step.
```
