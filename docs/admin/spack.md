# Managing Spack
Notes on creating/managing a spack envrionment at HPC{sup}`2`
Initially Conceived June 27, 2024, First Drafted July 8, 2024
Last updated July 8, 2024, by Huston Rogers (jhrogers)

**Revision History**
- v0: July 8, 2024, initial document
- v1: September 5, 2024, updated to include new paths, new config information in section 1

### Table of Contents
1. Installation and Global Configs 
2. Sourcing spack
3. Creating an Environment
4. Adding to an Environment
5. Modules
6. Cloning Environments
7. Cleanup <!--+ Appendix of Environments Created, to be updated as new environments are created/updated-->

## 1. Installation and Global Configs

### 1.1 Installing Spack
Spack is downloaded/pulled from the main github. We are focused on the development branch, because that is where 
package recipes are updated most often. There may not be a full release available when a package update is necessary

```
git clone https://github.com/spack/spack.git /apps/other/spack-devel
```

if it already exists
```
cd /apps/other/spack-devel ; git pull
```

### 1.2 Configuration Changes
Spack by default has some configuration that works, and some that we have changed for the HPC{sup}`2` environment.

#### 1.2.1 spackroot/etc/spack/config.yaml changes
search for the leading setting to find/update
```
all: "{compiler.name}-{compiler.version}/{name}-{version}-{hash}"

environments_root: /apps/other/spack-devel/environments

build_stage:
    - /tmp/$user/spack-stage
    - $user_cache_path/stage
```

#### 1.2.2 spackroot/etc/spack/packages.yaml
```
packages:
  slurm:
    externals:
      - spec: "slurm@hpc2"  
        prefix: /opt/slurm/
```

#### 1.2.3 spackroot/etc/spack/modules.yaml
```
modules:
  prefix_inspections:
    bin:
      - 'PATH'
    man:
      - 'MANPATH'
    share/man:
      - 'MANPATH'
    lib/pkgconfig:
      - 'PKG_CONFIG_PATH'
    lib64/pkgconfig:
      - 'PKG_CONFIG_PATH'
    share/pkgconfig:
      - 'PKG_CONFIG_PATH'
    lib64:
      - 'LD_LIBRARY_PATH'
    lib:
      - 'LD_LIBRARY_PATH'
  default:
    enable::
      - lmod
    lmod:
      exclude_implicits: true
      core_compilers:
      - 'gcc@11.3.1'
      hierarchy:
      - mpi
      hash_length: 0
      all:
        environment:
          set:
            '{name}_ROOT': '{prefix}'
```
### 1.3 Finding other externals
Run this to make sure you have all the other packages

```
spack external find
```

may need to update some, i.e. openssl's pkgconfig isn't always found consistently

```
  openssl:
    externals:
    - spec: openssl@3.0.1
      prefix: /usr
      extra_attributes:
        environment:
          prepend_path:
            PKG_CONFIG_PATH: /lib64/pkgconfig/
```

### 1.4 Notes about Config Changes

These are the base configs. They apply to ALL builds, but they apply **AFTER** the environment configs, detailed later

## 2. Sourcing spack

The spack configuration files are under /apps/other/spack-devel/etc/spack. To enable spack in your environment

```
source /apps/other/spack-devel/share/spack/setup-env.sh
```

Alternatively, 

```
module load spack/devel
```

which has an alias `fix-spack` that runs the above command. Making a modulefile that encompasses spack generates errors.

## 3. Creating an Environment.

### 3.1 Information
There exists one environment for every compiler+mpi+arch in a specific syntax:

\<COMPILER\>-\<VERSION\>-\<MPI\>-\<VERSION\>-\<ARCH\>

examples:

- gcc-11_3_1-nompi-x86_64_v3
	- this is the core environment
- gcc-11_3_1-mpich-4_2_2-x86_64_v3    
- gcc-11_3_1-openmpi-4_1_6-x86_64_v3  
- gcc-14_2_0-nompi-x86_64_v3                
- gcc-14_2_0-openmpi-4_1_6-x86_64_v3        
- gcc-14_2_0-mpich-4_2_2-x86_64_v3   
- oneapi-2023_2_4-nompi-x86_64_v3
- oneapi-2023_2_4-impi-2021_12_1-x86_64_v3
- oneapi-2024_2_1-nompi-x86_64_v3
- oneapi-2024_2_1-impi-2021_13_1-x86_64_v3
- gcc-11_3_1-nompi-skylake_avx512     

These environments are copied everywhere so that the master recipes are maintained, but only installed on specific hosts, i.e. Orion = skylake, Hercules = icelake, etc

### 3.2 Command Example
```
spack env create gcc-11_3_1-nompi-x86_64_v3
```

### 3.3 Ensuring compilers, etc. are set in the environment

Add:
/apps/other/spack-devel/environments/gcc-11_3_1-nompi-x86_64_v3/spack.yaml
```
  unify: when_possible
  include:
    config.yaml
    modules.yaml
  packages:
    all:
      compiler: ['gcc@11.3.1']
```

Include the compiler hash so that it's always the correct one, generic vs optimized. The core compiler doesn't need a hash

/apps/other/spack-devel/environments/gcc-11_3_1-nompi-x86_64_v3/config.yaml
```
config:
  install_tree:
    root:/apps/spack-managed-x86_64_v3-v1.0
    projections:
      all: "{compiler.name}-{compiler.version}/{name}-{version}-{hash}"
```
/apps/other/spack-devel/environments/gcc-11_3_1-nompi-x86_64_v3/modules.yaml
```
modules:
  default:
    roots:
      tcl: /apps/spack-x86_64_v3-v1.0/modulefiles
      lmod: /apps/spack-x86_64_v3-v1.0/modulefiles
```
- **NOTE**: 'v1.0' is in the root here because the spack tree itself is versioned, in the event that a spack change renders the tree unusable.

### 3.4 adding mpi

in the spack.yaml file, add, and include the hash under packages:
```
    mpi:
      require: [openmpi@4.1.6/awzguei]
```

### 3.5 fixing misbehaving compilers

in the spack.yaml file, for each spec, add the compiler

```
spec:
- package@version %gcc@14.2.0
```


## 4. Adding to an Environment

### 4.1 Information
We try to add only the latest versions when possible. To this end, spack info does a lot of help.

spack info netcdf-c

<Lots of info, including versions>

Additionally, be sure to activate the environment before adding to it (it should warn that you can't add without an environment)
```
spack env activate oneapi-2023_2_4-nompi
```

### 4.2 Command Example for adding a package
```
spack add netcdf-c +hdf4
```

### 4.3 Show the packages defined
```
spack find
```

### 4.4 Check that the package(s) are defined right.
```
spack concretize
```

### 4.5 Fixing boo boos
If a package doesn't concretize correctly, you would need to uninstall it and fix it. From above example, it tries to add intel-oneapi-mpi
and hdf5 as a dependency, and it may have done the wrong compiler. We don't want it to do mpi, for sure. We can add hdf5 beforehand, so that we only build one version

```
spack remove netcdf-c

spack add hdf5 +fortran +hl ~~mpi %oneapi@2023.2.4
spack add netcdf-c +hdf4 ~~mpi %oneapi@2023.2.4

spack concretize -f
```

### 4.5.1 Addendum, example in ticket 138580

```
#Env activate
spack env activate gcc-11_3_1-nompi-x86_64_v3
#Find the hash that needs to be updated
spack find --long netcdf-c
#Uninstall, including dependents
spack uninstall /zmfktng --dependents
spack uninstall hdf@4
#Edit the spack.yaml to have new options, move the spack.lock file to spack.lock.date(or dot bak)
spack concretize
spack install -j 4

and repeat for each arch target
```

### 4.6 Update/Install the new package(s)
```
spack install
```

## 4.7 Try and keep up with the packages that are added
```
intel-oneapi-runtime-2023.2.4-aosy4vtg6snij6ileqlcuqlc3u4pwf5i
cmake-3.29.6-suvclhyrpbpelwpfnazbsiyvvu3td53z
hdf5-1.14.3-twxi24pbgcej4azihuddjwui4q7dc5xp
netcdf-c-4.9.2-d53l6nm2hmncj5kzmwejkjzhkagmdv42
```

### 4.8 Repeat for anything else
```
spack add netcdf-fortran %oneapi@2023.2.4
spack add netcdf-cxx %oneapi@2023.2.4
spack add intel-oneapi-mpi %oneapi@2023.2.4

spack concretize -f

spack install
```
```
intel-oneapi-mpi-2021.13.0-k3d5ljngir6em3rpfpdcncow57mjpgqu
netcdf-cxx-4.2-vtj5zncsrgp5m3bv2l2u4tzysrkrfhyj
netcdf-fortran-4.6.1-3hzefwe33kpq7ydznbq4khbr442hpq26
```

### 4.9 sync from build node to prod if applicable (hercules, orion have separate build trees)

### 4.10 update the spack.yaml file to include a ticket number if applicable, or any extra notes
```
  - python@3.12.5+tkinter #HPC#139617, ALSO DID python3 -m ensurepip after
  - beast2@2.7.4 ^libbeagle+cuda #HPC#139362
```

### 4.11 you can use overwrite to make sure the installed packages are the clean ones.
```
spack install --overwrite -j 4
```
followed by examining what has a different touch date and removing the old folders

### 4.12 some compilers need bonus flags, i.e. intel, which are defined in the spack.yaml file of the env

## 5. Modules
modules are specific to environments, and should respect the config files

This means we can add stuff for specific files, and it take effect, i.e.:
```
modules.yaml for gcc-11_3_1-nompi-x86_64_v3:
modules:
  default:
    lmod:
      openmpi:
        environment:
          set:
            SLURM_MPI_TYPE: 'pmi2'
```

We can then safely:
```
spack module lmod refresh
```
second example: 
/oneapi-2023_2_4-nompi-x86_64_v3/modules.yaml
```
      intel-oneapi-mpi:
        environment:
          set:
            SLURM_MPI_TYPE: 'pmi2'
            I_MPI_EXTRA_FILESYSTEM: 'enable'
            I_MPI_PMI_LIBRARY: '/opt/slurm/lib/libpmi2.so'
            FI_PROVIDER: 'mlx'
```

## 6. Exporting (Copying/Cloning) Environment
Conveniently, since spack stores environments as simple files, it's possible to copy them to another location:
```
rsync -avH cluster1:/apps/spack-managed/spack-devel/var/spack/environments/<ENV-to-export>/spack.yaml cluster2:/tmp/<ENV-To-Import>-spack.yaml
spack env create <ENV-to-import> /tmp/<ENV-To-Import>-spack.yaml
```
This is also how we create the base environments. See the recipe files for how these are stored and maintained.

## 7. Cleanup
```
spack gc 
```
- Run the garbage collector after modulefiles, because it should only remove build-dependencies, not runtime. Theoretically.
- Garbage collection is env dependent. so it's somewhat safe to use.
