# rocHPL
## The Goal: Running HPL on MI210's
## Failed solutions
- wrap clblast as fortran blas
	- this kindof worked, but had runtime issues and was fundamentally flawed. standard HPL calls `dgemm_` in small chunks. each time it's called. I had to move buffers to the gpu and back this killed performance, putting it on par w/ cpu blas using the entire cpu.
- wrap rocblas as fortran blas
	- since rocblas is more optimized for MI210's. I tried to do the same as above. Never got it working.
- hpl-gpu
	- a long-since deprecated port of hpl that uses caldgemm to offload compute to a variety of platforms including opencl. while theoretically possible, refactoring necessary for such an out of date software makes this a no-go
- rocHPL
	- while the ideal option as far as hardware is concerned, it's build system would not work no matter what I tried. I even tried in a container with ideal conditions and it still failed.

## The Situation
I decided to dig deeper into why the configure stage for rocHPL was failing. I build my own spack tree independent of system rocm installs with all the necessary dependencies to eliminate that issue. I finally got past the dependency checking stage only to be met with `hip_add_executable() unknown verb` or the sort. It turns out that amd used to provide that routine in hip's cmake file, but deprecated it a few years ago. With very little knowledge of cmake, i was hesitant to try to refactor it's replacement. Finally, emboldened by the tool of generative AI, I thought I'd give it a try. 

## The Solution
The required changes are included in my internal repo [here](https://gitrepo.hpc.msstate.edu/tah568/rocHPL)

### Things to note
- hip_add_executable() has been deprecated. The replacement is the standard add_executable but now hip is treated as it's own language. I also removed the project's support for cloning and building it's own MPI stack. 