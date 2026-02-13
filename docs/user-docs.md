# Getting Started

## Accessing Muscadine
### Method One: Titan
First log into Titan at `titan.hpc.msstate.edu`. Use the following template if needed:

```bash
ssh <NetID>@titan.hpc.msstate.edu
```

After entering your HPC password and doing the 2FA push, you can proceed to logging into Muscadine. Muscadine only has one node facing the HPCC network which is `muscadine-node-1`, so that is the one we'll want to ssh into. Use the following template if needed:

```bash
ssh <NetID>@muscadine-node-1.hpc.msstate.edu
```

Repeat the authentication process from logging into Titan and you'll be set.

### Method Two: HPC VPN
Method Two involves logging into the HPC{sup}`2` VPN. Go to [https://servicedesk.its.msstate.edu](https://servicedesk.its.msstate.edu/TDClient/45/Portal/KB/?CategoryID=80) and download the Cisco AnyConnect VPN by following ITS's guide. Once that is completed, launch the VPN and connect to `vpn.hpc.msstate.edu`. It will then ask for your Group (HPC2), Username (your NetID), Password (your HPC Password), and Duo 2FA Passcode (get from duo app or enter 'push' to recieve a push challenge notification). Once you are connected, you can ssh directly into muscadine's head node `muscadine-node-1`. Use the following template if needed:

```bash
ssh <NetID>@muscadine-node-1.hpc.msstate.edu
```

---

## Muscadine's Storage
You are provided a decently fast and shared home directory. This is the location you start after connecting into Muscadine. You are welcome to keep stuff in your home directory, but there are also `/scratch` and `/work` that you can use.

`/scratch` is build off of the BeeGFS and is the most performant storage solution for Muscadine. The downside is that there is limited storage, thus there will be wipes of old files every now and then whenever it gets near capacity.

`/work` and `/home` are based on the same storage solution so they have the same speed. The difference is that `/work` is intended for project data such as datasets, AI models, output files, etc. If your program intakes data or outputs data, that data should be stored in `/work`.

| Mount      | Total Capacity | Speed      | Quota | Description                                                                                                                                                  |
| ---------- | -------------- | ---------- | ----- | ------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| `/scratch` | 2.2TB          | ***FAST*** | None  | Scratch is to be used for active data or programs that are not vital. ***SCRATCH WILL BE PERIODILY WIPED***                                                  |
| `/home`    | 20TB           | *quick*    | 10GB  | Home is where your and other user's personal directories live. Only put binaries, scripts, or small things in here. Capacity shared with `/work` and `/apps` |
| `/work`    | 20TB           | *quick*    | 500GB | Work is where datasets and large quantities of input/output data is to be stored. Capacity shared with `/home` and `/apps`.                                  |

```{note}
The speeds are intentionally vague. Feel free to benchmark them yourself
```
```{warning}
Do not put anything you want to keep for longer than a day in `/scratch` as all files are subject to deletion to preserve free space. 
```

---

## Using Muscadine
### Getting a node allocation
Say you want to build a given program. You could build it on the head, but that can interfere with other people doing the same or running a program on the head. An alternative to that is submitting a Slurm request to get a node for building purposes. Run the following command to reserve a node for a bash terminal.

```bash
srun -c16 --pty bash
```

### Available modules
Modules are software stacks such as GCC, Python, OpenBLAS, or ROCM that we provide that are optimized for the cluster its on. A list of all available modules can be seen by running `ml avail`. Here is a table of all the modules provided by default. A few more modules can be made available after loading a compiler like `aocc` or `gcc`. 

```
-------------------- /apps/spack-zen4-v1.0/modulefiles/Core -----------------
   amdblis/5.0       hip/6.2.4          python/3.12.5             rust/1.81.0
   aocc/5.0.0        hipblas/6.2.4      rocblas/6.2.4             sqlite/3.34.1
   apptainer/1.3.4   miopen-hip/6.2.4   rocm-cmake/6.2.4
   contrib/0.1       openmpi/5.0.5      rocm-device-libs/6.2.4
   gcc/14.2.0        python/3.11.9      rocsolver/6.2.4
```

In order to load a module, you can run `ml load <moduleName>`. Tab complete does work with ml and we encourage you use tab complete as it can also list several different versions we provide for each module, most will only have one though.

### What do I do if there is a program I want?
The simple answer is to build it from their source repository. The complex answer is to build it from their source repository. We do not install tools or programs for users. That is the user's job of using the cluster. After building a program, you should get a binary executable. If you want to make binaries you make accessible by you anywhere on the system, complete the following steps.

1. After building your binary, create a `.bin` in your home directory by running `mkdir ~/.bin`.
2. Now move that binary into the `.bin` directory.
3. Lastly, you'll need to add the path of that directory to your environment.
4. Edit `~/.zshrc` adding `export PATH="$PATH:$(HOME)/.bin` to the end.
```{note}
NOTE: *You only need edit .zshrc once.* 
```

Users can submit a request to add something they have built to `/apps/contrib` for other users to use as well. 

---

## Using Slurm
### Common Slurm commands
#### `sinfo`
`sinfo` and is a command that lets you see the cluster status. I have ran `sinfo` on Muscadine during a low usage time so all the nodes are idle. It is *highly* recommended to check this before submitting jobs, specifically greater than half a node in size.
```bash
╭─odh49@muscadine-node-1 ~
╰─$ sinfo
PARTITION  AVAIL  TIMELIMIT  NODES  STATE NODELIST
muscadine*    up   infinite      5   idle muscadine-node-[1-5]
```

#### `squeue`
`squeue` is a command that allows you to see any current jobs running on the cluster. This is very useful if you want to monitor if your job is running/waiting for resources along with its Job ID. I ran it on Muscadine while I have my meta-8B.sh running.
```bash
╭─odh49@muscadine-node-1 ~
╰─$ squeue
    JOBID PARTITION     NAME     USER ST       TIME  NODES NODELIST(REASON)
     3449 muscadine meta-8B.    odh49  R       0:18      1 muscadine-node-2
```

#### `scancel`
`scancel` kills a Slurm job. This is useful if you realize there was a mistake in your SBATCH file, made a change before it finished, or if it was an interactive job that froze. Here I used the Job ID from my job previously shown to kill the model running.
```bash
╭─odh49@muscadine-node-1 ~
╰─$ scancel 3449
```

#### `sacct`
`sacct` is like `squeue` except it shows all Slurm jobs that have been ran, is running, or errored out solely by you. This is very useful when looking at several jobs and seeing how its being handled. Here is the output of `sacct` with a few things running, ran, and failed.
```bash
╭─odh49@muscadine-node-1 ~/Demos/helloWorld
╰─$ sacct
JobID           JobName  Partition    Account  AllocCPUS      State ExitCode
------------ ---------- ---------- ---------- ---------- ---------- --------
3449         meta-8B.sh  muscadine     sighpc          2 CANCELLED+      0:0
3449.batch        batch                sighpc          2  CANCELLED     0:15
3449.0        llama-cli                sighpc          2  CANCELLED     0:15
3450          llama-cli  muscadine     sighpc          2     FAILED      0:2
3450.0        llama-cli                sighpc          2  CANCELLED      0:2
3451         meta-8B.sh  muscadine     sighpc          2    RUNNING      0:0
3451.batch        batch                sighpc          2    RUNNING      0:0
3451.0        llama-cli                sighpc          2    RUNNING      0:0
3452         helloWorld  muscadine     sighpc         48    RUNNING      0:0
3452.batch        batch                sighpc         48    RUNNING      0:0
3452.0        hello_mpi                sighpc         48    RUNNING      0:0
```

#### `srun`
`srun` is the command you'll use to run a parallel program via Slurm. For the most part, you'll use this within an SBATCH script described in Section 5.3. But if you want an interactive session (eg. `srun -c2 --pty bash`). Using `srun` is also useful when testing code on a small scale before fully implementing the SBATCH for it. Check out https://slurm.schedmd.com/srun.html for a full list of arguments to use with `srun`.


### Building a SBATCH script
SBATCH Scripts are what are used by Slurm to better allocate/distribute your program. Below is a template you can use for nearly any Slurm job.

```SBATCH
!/bin/bash

 Slurm Specific

SBATCH --job-name="my_name"            #job name
SBATCH --nodes=1                       #number of nodes
SBATCH --ntasks-per-node=16            #number of tasks per node
SBATCH --cpus-per-task=1               #number of physical cores per task
SBATCH --mem=4G                        #max memory allocated 
SBATCH --time=0-1:00:00                #max runtime (D-hh:mm:ss)
SBATCH --output=my_run.log             #output name


 Muscadine Specific

ml load <neededModules>


 Your Application

srun <application>
```

Here is an example of a SBATCH Script used for a Hello World spanning across 2 nodes, 48 threads each, using the openmpi/5.0.5 module, and the binary I made.

```SBATCH
!/bin/bash

 Slurm Specific

SBATCH --job-name=helloWorld           #job name
SBATCH --nodes=2                       #number of nodes
SBATCH --ntasks-per-node=48            #number of tasks per node
SBATCH --cpus-per-task=1               #number of cpus per task
SBATCH --mem=8G                        #max memory allocated
SBATCH --time=0-1:00:00                #max runtime
SBATCH --output=helloWorld-%j.out      #output name


 Muscadine Specific

ml load openmpi/5.0.5


 Your Application

srun ./hello_mpi
```

You can submit the SBATCH Script to Slurm via the following command:

```bash
sbatch <sbatchScript>
```
