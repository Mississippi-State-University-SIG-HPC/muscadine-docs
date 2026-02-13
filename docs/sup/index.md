# Supporting Software
Clusters are great, but on their own, they can accomplish very little. A cluster is essentially a collection of individual machines connected through a network, and without proper coordination, each node is just another independent computer. To make them function as a unified system capable of tackling a single large task, a significant amount of supporting software is required. This includes job schedulers, resource managers, distributed file systems, and communication frameworks such as MPI (Message Passing Interface), which allows processes on different nodes to exchange data efficiently. In other words, the real power of a cluster comes not from the hardware itself, but from the software stack that orchestrates and integrates it into a cohesive computing environment.

```{toctree}
:maxdepth: 1
:caption: Contents
:glob:

mpi-basics
*
```
