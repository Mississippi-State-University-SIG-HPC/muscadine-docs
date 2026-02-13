# Introduction
Muscadine is MSU Special Interest Group in High Performance Computing (SIG-HPC)'s cluster graciously gifted by Dell to the 2024 GIGADAWGS team. Muscadine's purpose is to be a student development cluster, teaching students at MSU skills in HPC.

---

## Muscadine Specs
### Hardware
#### Compute:
- 5x Nodes:
	- 1x AMD EPYC Genoa 9454P 48 core CPU
	- 12x 16 GB DDR5 4800MT/s (192GB total)
	- 1x Nvidia/Mellanox HDR 200Gbit/s HCA
	- 1x AMD Radeon Instinct MI210 (64GB) GPU

#### Storage:
- BeeGFS
	- 5x BOSS-N1 RAID-1 Arrays
	- mounted on `/scratch`
	- 2.2TB Distributed Filesystem
- PureStorage Flashblade
	- Clustered flash array storage appliance
	- 6 16TB blades for 96TB raw storage
	- mounted on `/home`, `/work`, and `/apps`
	- 48TB usable space (w/ 100% redundancy)

#### Networking:
- HDR Infiniband
	- 200Gbit/s RDMA
	- Used for:
		- Performant Cluster Communication
		- RDMA BeeGFS
- Gigabit Ethernet
	- 1Gbit/s
	- Used for:
		- NFS Storage
		- scheduling
		- admin
		- monitoring

---

### Architecture
Muscadine uses *AMD EPYC Genoa 9454P* running AMD's Zen 4 architecture. This architecture divides the CPU's cores into groups called *core complexes* (CCX) which share L3 cache. CCXs also group into dies called CCDs. CCXs act as NUMA partitions and interface with a I/O Die which handles communication between NUMA partitions and the rest of the system. There are 12 memory controllers for this generation, meaning there are 3 per CCD hardware defined. 

Specifically for Muscadine, there are 6 cores per CCX, 2 CCX per CCD, and 4 CCD per node. Additionally, each memory controller has its own 16GB memory DIMM, so most programs should optimize to the closest memory. Slurm is designed to be NUMA aware and our provided compilers are optimized for our architecture. More information on using architecture aware Slurm submissions, refer to [**Advanced Topics → Slurm Tips & Tricks**](/adv/slurm.md).

- CPU Specifications: [AMD EPYC™ 9454P](https://www.amd.com/en/products/processors/server/epyc/4th-generation-9004-and-8004-series/amd-epyc-9454p.html)
- GPU Specifications: [AMD Instinct MI210](https://www.techpowerup.com/gpu-specs/a100-sxm4-80-gb.c3746)
- Zen 4 White Paper: [Zen 4 White Paper](https://www.amd.com/en/products/processors/server/epyc/4th-generation-architecture.html)

---

### Benchmarks Leaderboard
To promote a sense of competition. Here we will host a leaderboard of the best scores of various HPC benchmarks. In order to get your score verified, email your entire config and output file to the current student sysadmin. 

#### HPL:

| Rank | Owner               | 1-Node        | 5-Node        | CPU/GPU |
| ---- | ------------------- | ------------- | ------------- | ------- |
| 1st  | Drew Helgerson      | 19.06 TFlop/s | 104.7 TFlop/s | GPU     |
| 2nd  | Oliver Higginbotham | 2.371 TFlop/s | 11.62 TFlop/s | CPU     |
%%| 1st  | Drew Helgerson      | 2.392 TFlop/s | 11.81 TFlop/s | CPU     |%%

See [Tuning HPL](/adv/hpl.md) to get started

#### HPCg:
Nobody has submitted scores for HPCg