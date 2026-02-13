# MPI Programming Basics

This guide walks through writing, building, and running a simple **MPI ping** program using **SLURM**.

This program is designed to show you the bare minimum to get starting writing with OpenMPI. MPI is a large rabbit hole you can go down in order to eek out more performance of your cluster. Things like unbuffered send and receive, broadcasting, reducing, etc. are all possible with MPI. 
Later on, we'll look at calculating digits of Pi using MPI.

---

## Overview

We'll create a two-process program where **rank 0** sends a message to **rank 1**,  
and **rank 1** sends it back — like a simple "ping-pong" test.

---

## Writing the Program
Save the following as `ping.c`:

```c
#include <mpi.h>
#include <stdio.h>
#include <string.h>

int main(int argc, char** argv) {
    MPI_Init(&argc, &argv);

    int rank;
    MPI_Comm_rank(MPI_COMM_WORLD, &rank);

    const int tag = 0;
    char msg[32] = "ping";
    MPI_Status status;

    if (rank == 0) {
        MPI_Send(msg, strlen(msg) + 1, MPI_CHAR, 1, tag, MPI_COMM_WORLD);
        MPI_Recv(msg, sizeof(msg), MPI_CHAR, 1, tag, MPI_COMM_WORLD, &status);
        printf("Rank 0 received: %s\n", msg);
    } else if (rank == 1) {
        MPI_Recv(msg, sizeof(msg), MPI_CHAR, 0, tag, MPI_COMM_WORLD, &status);
        printf("Rank 1 received: %s\n", msg);
        strcpy(msg, "pong");
        MPI_Send(msg, strlen(msg) + 1, MPI_CHAR, 0, tag, MPI_COMM_WORLD);
    }

    MPI_Finalize();
    return 0;
}
```

---

## Building the Program
Use your MPI compiler wrapper:

```bash
# Load the gcc build of openmpi into our environment
ml gcc openmpi
# compile our source into the 'ping' binary
mpicc -o ping ping.c
```

This ensures proper MPI headers and libraries are included.

---

## Running the Program with SLURM
Create a batch script `ping.sh`:

```bash
#!/usr/bin/env bash

# set our job name
#SBATCH --job-name=mpi-ping
# we're not picky about where our ranks live, so we'll just specify we need 2 of them
#SBATCH --ntasks=2
# if it takes longer than a minute to run this simple application, we've got a bigger problem
#SBATCH --time=00:01:00

# finally launch the executible accross our allocation
srun ./ping
```
```{note}
it's important to include the `./` before our executable since ping exists in our path, but is not written for MPI
```


Then submit the job:

```bash
sbatch ping.sh
```

Alternatively, you can run interactively on a SLURM allocation:

```bash
srun -n2 ./ping
```
---

## Example Output

```text
Rank 1 received: ping
Rank 0 received: pong
```

---

## Notes
- `mpicc` ensures portability across MPI implementations (OpenMPI, MPICH, etc.).
- SLURM’s `srun` automatically launches MPI ranks on allocated nodes.

---

## See Also

- [MPI Tutorial (mpitutorial.com)](https://mpitutorial.com/)
- [SLURM sbatch documentation](https://slurm.schedmd.com/sbatch.html)