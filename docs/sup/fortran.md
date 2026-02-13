# What Even is Fortran?
## Intro and Getting Started
### So What is Fortran?
Fortran is a high-performance programming language built for fast and reliable numerical computing. It’s statically and strongly typed, which means many issues are caught during compilation, and the compiler can optimize your code for speed. Even though the language has a clean and compact syntax, it supports several programming styles—procedural, array-based, object-oriented, and more. Fortran also has built-in tools for parallel computing, making it a strong choice for scaling applications on clusters and supercomputers. While it’s one of the oldest programming languages, it continues to evolve, with the latest standard being Fortran 2023. Modern compilers and tools like the Fortran Package Manager (fpm) make it easy to develop and maintain Fortran projects today.

### What is Fortran Used For?
Fortran is widely used in science and engineering, especially in fields that rely on heavy numerical calculations. It powers applications like weather and ocean modeling, fluid dynamics, applied math, statistics, and quantitative finance. The language shines when working with large numerical datasets and array operations where performance matters most. Many HPC benchmarks and supercomputer workloads still rely on Fortran because it’s fast, stable, and well-suited to parallel computing. If you need to crunch a lot of numbers efficiently, Fortran is often one of the best tools for the job.

## Compiling Fortran Code
Compiling Fortran programs is straightforward, whether you're working with a simple file or a larger project. Below are some common commands and best practices to help you get started. 

### Basic Compilation
For small projects or single files, you can compile directly from the command line: 

Using gfortran (GNU Fortran)
```bash
gfortran hello.f90 -o hello
```

```{note}
-o specifies the name of the output executable.
```

### Useful Compiler Flags
When working on larger or performance-sensitive applications, adding flags can help with speed, debugging, or parallelization: 

`-O3`
: Enables many common high-level optimizations for better performance; better when AVX instructions are present.

`-fopenmp`
: Enables OpenMP for parallel programming (See [OpenMP](/adv/openmp.md))

`-g`
: Includes debugging information for tools like gdb

`-Wall`
: Shows common warnings to catch errors early

`-march=znver4`
: Enable all instructions available on Muscadine (such as AVX512). Use `-march=native` if you're unsure which architecture other machines are.

Example: 
```bash
gfortran -O3 -fopenmp -Wall -march=znver4 mycode.f90 -o mycode
```
```{note}
Compiler optimization flags may not always increase performance and can sometimes lead to performance regression. Experimentation is encouraged.
```

### Example Makefile for Larger Projects
For multi-file projects, a Makefile simplifies the build process and keeps everything organized. Here's a minimal example: 
```bash
FC = gfortran
FLAGS = -O3 -fopenmp -Wall
SRC = main.f90 module.f90 solver.f90
OBJ = $(SRC:.f90.o)
TARGET = myprogram

all: $(TARGET)

$(TARGET): $(OBF)
	$(FC) $(FLAGS) $(OBJ) -o $(TARGET)
	
%.o: %.f90
	$(FC) $(FLAGS) -c $<

clean:
	rm -f *.o *.mod $(TARGET)
```

Build your project by simply running: 
```bash
make
```

And clean up generated files with: 
```bash
make clean
```

This setup lets you easily recompile only the files that changed, making your workflow much faster and more efficient. 

## Using Libraries
Fortran integrates smoothly with high-performance math and I/O libraries, making it ideal for scientific and engineering applications. Many clusters come with preinstalled libraries like LAPACK, BLAS, MKL, ScaLAPACK, and HDF5. Linking against these can dramatically improve performance and expand functionality. 

### Linking LAPACK and BLAS
LAPACK and BLAS are standard linear algebra libraries widely used in scientific computing. If they're installed on your system, you can link them directly during compilation: 
```bash
gfortran myprog.f90 -llapack -lblas -o myprog
```

If the libraries are installed in a non-standard location, which they are, you will need to include additional flags such as: 
```bash
gfortran myprog.f90 -o myprog \
	-L/path/to/lib \
	-I/path/to/include \
	-llapack -lblas 
```

Muscadine offers modules to load these libraries, e.g.: 
```bash
module load gcc
module load openmpi
module load openblas
```

In that case, the compile line would look like:
```bash
gfortran myprog.f90 -o myprog \
	-L$OPENBLAS_ROOT/lib \
	-I$OPENBLAS_ROOT/include \
	-llapack -lblas
```

### ScaLAPACK and HDF5
Many HPC clusters also support additional libraries like ScaLAPACK and HDF5 for distributed linear algebra and data handling. 


## Best Practices & Tips
A little structure goes a long way when working on Fortran projects, especially in an HPC environment. Good organization, version control, and thoughtful I/O choices can save time and help your code scale smoothly on a cluster. 

### File Organization and Build Scripts
Keep your project directory organized so it's easy to navigate and build. Using a Makefile keeps compilation consistent and avoids having to remember long command lines. It's also easier to integrate new files without modifying build commands manually. 

### Using Version Control (Git)
Version control makes collaboration and backup much easier. Git is the most common tool and works well even for small projects. See [Using Git](/adv/git.md) for more details.

### Large File I/O 
For large simulations (common with Fortran), file I/O can become a performance bottleneck. A few practical tips: 
- Prefer binary files for large datasets - they are smaller and read/write faster than text. 
- Use unformatted I/O or libraries like HDF5 for structured binary storage.
- Compress or clean up old output files regularly to save space. 
- Keep output structured and labeled with timestamps or run IDs.
- Utilize MPIIO to help streamline writes from multiple nodes.

### Memory Management
Efficient memory management is crucial in Fortran, especially for large simulations or parallel jobs. Always allocate only what you need and deallocate when you're done with a variable. Forgetting to free memory can lead to unnecessary usage or, in larger runs, job failures due to memory limits.
```bash
real, allocatable :: A(:,:)

! Allocate memory
allocate(A(1000,1000))

! ... use A for computation ...

! Free memory when A is no longer needed
deallocate(A)
```
