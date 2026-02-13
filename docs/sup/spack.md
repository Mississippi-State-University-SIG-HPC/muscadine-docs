# Building Software with Spack
Resources: [spack](https://spack.readthedocs.io/en/latest/getting_started.html)

Spack is an HPC-focused, source-based package manager. Spack is helpful for both building software as well as dependencies for manually-built software. 

## Foreword
Much care and consideration goes into curating a given cluster's site-wide apps tree. It is imperative that users do not attempt to rebuild some of this software.

## Installation
### Initial setup
Spack is a user-land python suite that can be installed almost anywhere. Installing is easy.
```bash
https://spack.readthedocs.io/en/latest/getting_started.html /path/to/install
```

### Environment Setup
Each time you open a new shell, Spack needs to be loaded into your environment. do this with:
```bash
. /path/to/spack/share/spack/setup-env.sh
```

### Configuring Spack
As stated in the [Foreword](#foreword), it's imperative some software not be rebuilt. The best way to ensure this, as well as relieve the workload of rebuilding some dependencies, is to select an 'upstream' in spack. this way, spack uses what's already in the site-wide apps tree.
```bash
spack config add upstreams:apps:install_tree:/apps/spack-managed-zen4-v1.0
```

## Usage
Using spack is quite simple, the general workflow looks like this:
1. create an environment
2. identify a spec(package) to add
3. investigate what the spec line should be
4. add the spec to the environment
5. concretize
6. install (build)
7. load (optional)

### Create an Environment
Environments are how spack compartmentalizes its installed packages. environments can share packages but cannot share specs. reducing the size of environments is how spack keeps the computational complexity of solving the dependency tree low. It's advised to create a new environment for each target piece of software or project.

```bash
spack env create my-environment
spack env activate my-environment
```

### Identifying a spec
A spec refers to the target software you want spack to install. examples include 'hpl', 'openblas', 'chameleon', or your favorite version of mpi. For each possible spec, spack maintains a recipe for what this package needs and how to build it.

For example: say I want to build the dependencies for HPL. I'd need a compiler, an mpi, and a blas library. An example would be 'gcc@15', 'openmpi@5.0.8', and 'amdblis'

```{note}
 Versions are specified with an @. They aren't required but allow you to be specific.
 ```

### Writing a spec line
Spack is very specific about how it builds its software. You can choose to be as specific or vague about your specs. Being vague allows spack to choose the best (easiest to solve) specs.

You can view all the possible spec options for a package by issuing
```bash
spack info <package>
```

This will show all the info provided about a package recipe including possible verions, build systems, configuration options, etc.

#### The spec line
A normal spec line looks like this
```bash
hpl@2.3 %gcc@15 ^openmpi ^amdblis
```

Let's break that down.

`hpl@2.3`
: here we're saying we want spack to install hpl version 2.3

`%gcc@15`
: % tells spack this is the compiler we want
: Here we want any subversion of gcc 15

`^openmpi`
: ^ means we want our package to use this package as a dependency. In this case we want any openmpi to satisfy our mpi dependency

`^amdblis`
: Another dependency; use 'amdblis' to satisfy our blas dependency.

### Adding the spec
We can supply the spec in one of two ways, either by running the `spack add` command, or by adding it to the module config.

#### `spack add`
```bash
spack add hpl ^amdblis ...
```

#### `spack config edit`
this command will open the environment's yaml file in an editor for you to modify. add a spec by creating a newling under `specs` with your spec line.

### Concretize
Concretization is very simple, it's the part when spack solves the dependency tree and 'sets the specs in concrete'. Simply run it with 
```bash
spack concretize
```

After waiting a bit (a while if it's your first time), you'll be presented with a rudimentary graph of the dependency tree of your chosen specs. note that all packages have a complete spec line and nothing is to chance. It's at this point that you should look over the specs and make sure that spack plans to do what you want.

```{note}
It's at this point that spack can have issues. Solving the dependency tree can be difficult and sometime impossible. If the process takes more than ~5 minutes or fails, back up and try again. If you are having trouble, see [Getting Help](/getting-help.md).
```

### Install
Good news! The hard work is mostly done. at this stage, spack does all the work (and it's a lot).

```bash
spack install -jX # where X is desired cores
```

Building an entire dependency tree from source can take a long time. be patient. Spack will give status updates as it goes. Often times if spack seems to be doing nothing, it's working, just give it time.

### Load
Once your packages are finally installed. it's up to you to use them. by default, spack maintains what it calls 'views'. these are places where it tries to make relavent files available automatically for your environment. for some build systems, this works and that's all you need. for others, it may be necessary to `spack load package`. This is the equivalent of `module load an-app` for your individual spack tree; it loads the necessary environment variables for your build system to pick up on.

#### `spack location`
Some build systems can be quite stubborn. in this case, it may be necessary to manually specify where packages are located. An easy way to find this information is to use `spack location -i my-package`. this will return the install prefix of that package. Keep in mind that library files are in `/lib` or `/lib64`, headers are in `/include`, and binaries are in `/bin` relative to this path.