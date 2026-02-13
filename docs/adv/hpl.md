# Building and Tuning HPL

This guide walks through building, running, and *tuning* **HPL (High Performance Linpack)** on a Slurm cluster such as Muscadine. It assumes:

* You already have a functioning cluster ✅
* functional MPI ✅
* Slurm works ✅
* You are comfortable compiling software and reading performance numbers

---

## What HPL Actually Measures

HPL solves a dense system of linear equations using LU decomposition. In practice, HPL performance is dominated by:

* **DGEMM performance** from your BLAS
* Memory bandwidth and NUMA effects
* MPI latency and bandwidth
* Process/thread placement

If DGEMM is slow, HPL will be slow. Everything else is secondary.

---

## Choosing a BLAS Library

This is the single most important decision.

### Common BLAS Options

#### Intel MKL

- Usually the best performance on Intel CPUs
- Excellent threading and NUMA behavior
- Proprietary but free to use

Pros:

- Usually the fastest
- Minimal blas-specific tuning required

Cons:

- Muscadine has AMD Processors
- Closed source

We'll consider other options for Muscadine

#### OpenBLAS

- Open source
- Works everywhere
- Performance varies widely by architecture

```bash
module load openblas
```

Pros:

- Portable
- Easy to build

Cons:

- Threading is less predictable
- NUMA handling is weaker

#### (AOCL-)BLIS

- Modular, modern BLAS
- Very strong on AMD EPYC

```bash
module load amdblis
```

Pros:

- Excellent EPYC performance
- Cleaner threading model than OpenBLAS

Cons:

- Limited hardware support

#### Vendor BLAS (AOCL, Cray LibSci, etc.)

If your vendor provides a tuned BLAS, use it. They exist for a reason.

### Notes on Multi-threading

If you're unsure of the difference between Multi-Processing and Multi-Threading, read [this](/adv/mp-mt.md) first

All spack-managed blas libraries (ones you use `module load` to use) have multi-threading **DISABLED**. This means if you'd like to experiment with using hybrid-parallelism to gain more performance, you'll need to build your blas library of choice, following their documents to enable it.

---

## MPI Implementation

If blas is the single most important performance factor of HPL, Inter-process communication speed and, more importantly, latency, are the second.

Common choices:

* OpenMPI
* MPICH
* Intel MPI
* Cray MPI

Key requirements:

* Good support for your interconnect
* Correct PMI/PMIx integration with Slurm

Muscadine uses a 200Gbit/s Mellanox Infiniband Interconnect. MPI implementations that support this are: Intel MPI, OpenMPI, and MPICH.

Currently, only OpenMPI is provided to Muscadine users. It has compiled in support for slurm, pmi, infiniband, AMD, etc.

```bash
module load openmpi
```

```{warning}
Please do **NOT** attempt to build/install a different version of MPI. These builds require a rich understanding of the hardware and how it's configured in order to build correctly. We've already done the work for you, please use it.
```


---

## Building HPL

### Get the Source

```bash
curl -LO https://www.netlib.org/benchmark/hpl/hpl-2.3.tar.gz
tar xf hpl-2.3.tar.gz
cd hpl-2.3
```

### Configure the Makefile

Since version 2.3 (the latest as of writing), HPL can either use static Makefiles or GNU Autotools. Since autotools is alot simple and somewhat more consistent, we'll be using that.

```bash
./configure --prefix=$PWD/hpl-demo \
  CC=mpicc \
  LDFLAGS="-L$OPENBLAS_ROOT/lib -L$OPENMPI_ROOT/lib" \
  CPPFLAGS="-I$OPENMPI_ROOT/include" \
  CFLAGS="-O2 -march=znver4 -mtune=znver4 -DHPL_PROGRESS_REPORT"
```

### Build

```bash
make -j$(nproc)
make install
```

Binary appears as:

```bash
hpl-demo/bin/xhpl
```

---

## Hybrid MPI + OpenMP

Modern clusters rarely run optimally with just one rank per core. Shared memory can lead to increased utilization and efficiency. At scale, thousands to tens of thousands of ranks communicating over the fabric can significantly reduce performance.

Instead, it's preferable to use a hybrid approach.

### Why Hybrid?

- Reduces MPI rank count
- Improves cache reuse
- Reduces communication overhead

Typical strategy:

- 1 MPI rank per socket
- OpenMP threads = cores per socket

Modern Chiplet architecture strategy (Muscadine is one):

- 1 MPI rank per NUMA domain (CCX in the case of Muscadine)
- OpenMP threads = cores per CCX

Example (8 CCX, 6 cores/CCX):

```bash
#SBATCH --ntasks-per-node 8
#SBATCH --cpus-per-task   6

export OMP_NUM_THREADS=6
export OMP_PROC_BIND=close
export OMP_PLACES=cores
```

```{note}
The keen-eyed who know their way around Muscadine might have noticed that {math}`6*8=48` and Muscadine has {math}`96` threads. AMD Epyc and many other processors support Symmetric Multi-Threading (SMT) otherwise known as Hyper-Threading. While this feature is available, HPL already saturates the pipeline enough to make SMT ***less*** efficient. The scheduler is smart enough to keep threads on their own core.
```

---

## HPL.dat Tuning

`HPL.dat` controls performance-critical parameters.

### Problem Size (N)

Problem size is the parameter you'll mess with the most. HPL is a very Memory Bandwidth-intensive application. the more memory you use, the more the LU algorithm with stripe across it, increasing bandwidth. Use to little, and you won't saturate your memory bandwidth, use too much, and you'll start swapping. A perfect *HERO* run balances the two.

Rule of thumb:

```
N ≈ sqrt(0.8 × total_memory_bytes / 8)
```

Example: 192 GB total RAM

```
N ≈ sqrt(0.8 × 192e9 / 8) ≈ 130k
```

Use as much memory as possible *without swapping*.

If you'd like to play around with it. I've created a handy [desmos calculator](https://www.desmos.com/calculator/h7g9c3pbwe).

---

### Block Size (NB)

Block size is how much of the matrix each chunk works on at a time. This value is very CPU architecture-specific. Calculating the precise value requires a deep knowledge of how the cores are laid out in the CPU. Thankfully, it's easier to exhaustively find this value. We know that NB needs to be a multiple of cores. Simply run a test for each multiple of cores between 100 and 400. 

Typical values:

* 128
* 192
* 256
* 384

---

### Process Grid (P × Q)

Choose a grid close to square:

```
P × Q = number of MPI ranks
```

Example: 16 ranks

```
P=4, Q=4
```

Rule:

* Q ≥ P
* Match network topology if possible

---

### Key Parameters Summary

```text
N    : As large as memory allows
NB   : 100-400%core-count (benchmark!)
P,Q  : Nearly square
```

---

## Slurm Job Script Example
### MPI only

```bash
#!/bin/bash
#SBATCH -N 4
#SBATCH --ntasks-per-node=48
#SBATCH --cpus-per-task=1
#SBATCH -t 00:30:00
#SBATCH -J hpl

module load openmpi
module load openblas

srun --cpu-bind=cores ./xhpl
```

### MPI + OpenMP

```bash
#!/bin/bash
#SBATCH -N 4
#SBATCH --ntasks-per-node=8
#SBATCH --cpus-per-task=6
#SBATCH -t 00:30:00
#SBATCH -J hpl

module load openmpi
module load openmp

export OMP_NUM_THREADS=$SLURM_CPUS_PER_TASK
export OMP_PROC_BIND=close
export OMP_PLACES=cores

srun --cpu-bind=cores ./xhpl
```

Verify placement:

```bash
srun --cpu-bind=verbose ./xhpl
```

---

## NUMA and Affinity

If NUMA is wrong, performance collapses.

Recommendations:

* Use `numactl --hardware` to inspect layout
* Align MPI ranks with sockets
* Bind OpenMP threads tightly

Example:

```bash
srun --distribution=block:block --cpu-bind=cores ./xhpl
```

---

## Benchmarking Strategy

Never trust a single run.

Suggested sweep:

* Fix N
* Sweep NB = {128,192,256,384}
* Try different P×Q layouts
* Record Gflop/s

Track:

* Per-node performance
* Scaling efficiency

---

## Interpreting Results

### Expected Efficiency

* 80–90% of theoretical peak is *excellent*
* 70–80% is common
* <60% usually indicates:

  - Bad BLAS
  - Bad affinity
  - Bad NB
  - too few or many N's

---

## Common Mistakes

- Using tiny N
- Ignoring NUMA
- Using default OpenBLAS builds
- Running one MPI rank per core

---

## Final Advice

HPL is an incredibly nuanced benchmark that requires practical knowledge of every facet of the hardware. Don't be discouraged if you cannot achieve the same scores as someone else. 

If you have any questions, see those listed in [Getting-Help](/getting-help.md)