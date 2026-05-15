# Monitoring Slurm Jobs

## Overview

Muscadine provides two ways to monitor your Slurm jobs:

- **`jobstats` CLI**: a command-line tool that prints a summary of CPU, memory, and GPU usage for any completed or running job.
- **Grafana Dashboard (via Open OnDemand)**: a time-series dashboard that is able to be viewed under the `Jobs` menu on Open OnDemand that shows how your job used resources throughout the job's lifetime.

---

## 1. Monitoring Jobs with Jobstats

### What it shows

`jobstats` pulls data from the cluster's Prometheus and Slurm's accounting records to give you a per-job resource efficiency report, including:

- CPU Utilization
- Memory Usage
- GPU Utilization
- VRAM Usage
- Wall Time Efficiency

### Basic usage

After submitting a Slurm job, keep note of the JobID. That is the only input `jobstats` needs to work correctly. This is how you can run the command:

```bash
jobstats <jobid>
```

**Example:**

```bash
jobstats 5555
```

The output would be something like this:

```
================================================================================
                              Slurm Job Statistics
================================================================================
         Job ID: 6035
   User/Account: NetID/GroupID
       Job Name: jobstats-demo
          State: COMPLETED
          Nodes: 1
      CPU Cores: 4
     CPU Memory: 4GB (1GB per CPU-core)
           GPUs: 1
  QOS/Partition: normal/muscadine
        Cluster: muscadine
     Start Time: Wed May 13, 2026 at 3:57 PM
       Run Time: 00:02:42
     Time Limit: 00:30:00

                         Overall Utilization
================================================================================
  CPU utilization  [|||||||||||                                    22%]
  CPU memory usage [||||||                                         12%]
  GPU utilization  [||||||||||||||||||||||||||||||                 60%]
  GPU memory usage [                                                1%]

                         Detailed Utilization
================================================================================
  CPU utilization per node (CPU time used/run time)
      muscadine-node-2: 00:02:22/00:10:48 (efficiency=22.0%)

  CPU memory usage per node - used/allocated
      muscadine-node-2: 530.8MB/4.3GB (132.7MB/1.1GB per core of 4)

  GPU utilization per node
      muscadine-node-2 (GPU 0): 60%

  GPU memory usage per node - maximum used/total
      muscadine-node-2 (GPU 0): 716.0MB/64GB (1.1%)
```

---

## 2. Viewing the Grafana Dashboard via Open OnDemand

For a full time-series view and graphs that show exactly how CPU, memory, and GPU behaved **minute-by-minute** from job start to job end, use the `Muscadine Slurm Job Stats` Grafana dashboard.

### Step 1: Log into the OOD portal

Navigate to the Muscadine Open OnDemand portal at https://muscadine-ood.hpc.msstate.edu, and log in with your MSU HPC NetID credentials.

```{note}
To visit the Muscadine OOD portal, you must be connected to the MSU HPC VPN. For more information, look here: {ref}`VPN setup steps <method-two-hpc-vpn>`
```

---

### Step 2: Open Job Stats Helper

From the top navigation bar, click:

```
Jobs → Job Stats Helper
```

This opens a new window to the Job Stats Helper app, which generates a Grafana dashboard for a specific JobID.

Enter the JobID for the job you want to view metrics for.

---

### Step 3: Generating the Dashboard

After typing your JobID, click `Submit Query`. Then, under **Link with detailed statistics**, click `Click here for stats`.

This will bring you to a pre-configured Grafana dashboard where you can view the entire lifetime metrics of that specific job.

---

### Step 4: Viewing the "Muscadine Slurm Job Stats" Dashboard

The dashboard opens pre-configured to your specific job ID and time range (from the moment the job started to when it ended, or the current time if it's still running).

**Panels included:**

| Panel | What it shows |
|---|---|
| **Job CPU Utilization (%)** | User, system, and total CPU usage as a percentage of allocated cores over time |
| **Job CPU Memory Utilization (%)** | RSS, cached, used, and total allocated memory over time, plus OOM failure events |
| **Allocated CPU Memory** | The total memory allocation as specified in your `sbatch` script (`--mem`) |
| **Allocated CPU Cores** | The number of CPU cores allocated to the job (`--ntasks` / `--cpus-per-task`) |
| **GPU Utilization (%)** | GFX engine activity percentage across all GPUs assigned to the job over time |
| **GPU Memory Utilization (%)** | VRAM consumption as a percentage of total VRAM across all assigned GPUs over time |
| **GPU Memory Utilization (MB)** | Raw VRAM consumption in MB across all assigned GPUs over time |
| **Memory Controller Utilization (%)** | UMC (Unified Memory Controller) activity (how hard the GPU memory bus is being pushed) |
| **GPU Clockspeed (MHz)** | Active GPU core clock frequency over the job's lifetime |
| **GPU Temperature (°C)** | Edge and junction temperatures for the assigned GPUs over the job's lifetime |
| **GPU Power Usage (W)** | Average package power draw in watts for the assigned GPUs over time |

**Additional Panels:**

| Panel | What it shows |
|---|---|
| **Node CPU Percentage Utilization** | System, user, IO-wait, and total CPU usage at the node level (independent of job cgroup accounting) |
| **Node Total Memory Utilization** | Full node memory breakdown (total, used, available, free, buffered, and cached) |
| **Local Disk R/W** | Read and write throughput (bytes/sec) per block device on the selected node |
| **Local Disk IOPS** | Read and write I/O operations per second per block device on the selected node |
| **NFS Stats** | NFS read/write request rate and metadata operation rate (reads and writes) over NFSv3 |
| **Infiniband Throughput** | Bytes per second received and transmitted over the node's Infiniband port |
| **Infiniband Packet Rate** | Multicast and unicast packet rates in both directions over Infiniband |
| **Infiniband Errors** | Link errors, downed links, congestion indicators, and discarded packets on the Infiniband fabric |

**The time range is automatically scoped to your job**, and you don't need to manually adjust it. The dashboard uses the job's start and end timestamps pulled from Slurm accounting.

---

## 3. Tips for Interpreting Your Job's Metrics

**CPU efficiency below 50%?**
Your job may be waiting on memory, I/O, or GPU transfers rather than doing compute work. Note that 50% is considered fully utilized for single-threaded code (since Slurm counts hyperthreads as cores). If your job isn't using multi-threading, efficiency near 50% is expected and healthy. Below that, check if you're over-allocating cores relative to what your code can actually parallelize.

**Memory usage near the limit?**
You're at risk of OOM kills on future runs. Request more memory or reduce your problem size per node.

**VRAM usage unexpectedly high?**
Check for memory leaks or unfreed allocations across iterations. The time-series view in Grafana is especially useful here. Look for a rising VRAM trend over the job's lifetime rather than a stable plateau.

---

## More Info

Jobstats Documentation: https://princetonuniversity.github.io/jobstats/
