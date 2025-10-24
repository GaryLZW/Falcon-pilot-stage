# Falcon-pilot-stage

This stores experiences from Falcon pilot stage 1.

## Compilation of aims binary.

Load modules.
```
module load intel-compilers/2025.1.1 impi/2021.15.0-intel-compilers-2025.1.1 imkl/2024.2.0
module load CMake/3.31.3-GCCcore-14.2.0
```
This is the content of my ```.cmake``` file. 
Due to the new intel compiler, mpif90 &rarr; mpifx; icc &rarr; icx; icpc &rarr; icpx
```
# Intel Compilers
set(CMAKE_Fortran_COMPILER "mpiifx" CACHE STRING "" FORCE)
set(CMAKE_Fortran_FLAGS "-O3 -ip -fp-model precise" CACHE STRING "" FORCE)
set(Fortran_MIN_FLAGS "-O0 -fp-model precise" CACHE STRING "" FORCE)
set(CMAKE_C_COMPILER "icx" CACHE STRING "" FORCE)
set(CMAKE_C_FLAGS "-O3 -ip -fp-model precise -std=gnu99" CACHE STRING "" FORCE)
set(CMAKE_CXX_COMPILER "icpx" CACHE STRING "" FORCE)
set(CMAKE_CXX_FLAGS "-O3 -ip -fp-model precise" CACHE STRING "" FORCE)
set(LIB_PATHS "/shared/apps/easybuild/x86_64/amd/zen4/software/impi/2021.15.0-intel-compilers-2025.1.1/mpi/2021.15/lib" CACHE STRING "" FORCE)
set(LIBS "mkl_intel_lp64 mkl_sequential mkl_core mkl_blacs_intelmpi_lp64 mkl_scalapack_lp64" CACHE STRING "" FORCE)
set(USE_MPI ON CACHE BOOL "" FORCE)
set(USE_SCALAPACK ON CACHE BOOL "" FORCE)
set(USE_LIBXC ON CACHE BOOL "" FORCE)
set(USE_HDF5 OFF CACHE BOOL "" FORCE)
set(USE_RLSY ON CACHE BOOL "" FORCE)
set(ELPA2_KERNEL "" CACHE STRING "" FORCE)
```

## Build python venv

```module load Python/3.10.4-GCCcore-11.3.0-bare```
I had to use this version of python, because ```QUIP``` does not support python 3.11 and above.

Unfortunately, the module ```intel-compilers/2025.1.1``` is bind to the ```GCCcore/14.2.0```. 
The other available intel compilers do not bind to ```GCCcore/11.3.0``` as well.


## Submission of jobs
+ Partition

  I can't access compute partition with
  ```
  #SBATCH -n 384
  #SBATCH -p compute
  #SBATCH --ntasks-per-core=1
  ```
  somehow.
  The number of nodes has to be specified.
  ```
  #SBATCH --nodes=2
  #SBATCH -n 384
  #SBATCH --ntasks-per-core=1
  #SBATCH -p compute
  ```
  
   
+ Memory

  ```#SBATCH --mem=0``` is not treated as requiring all memory, but some predefined memory? 
  
  I remember run into memory error with this. 

  I had to specify ```--mem=200G``` to make my density matrix restart workflow run, though each node have more memory than that.
  
  I'm not sure why restarting from density matrix ```.csc``` files in python requires so much memory, but I have a similar problem on hawk.

+ Run aims

  ***All denpendencies needs to be specified.*** Intel compiler; MPI; MKL
  
  Specify PMI library path. 
  ```export I_MPI_PMI_LIBRARY=/usr/lib64/libpmi.so```

  Or it gives this error: ```MPI startup(): PMI server not found. Please set I_MPI_PMI_LIBRARY variable if it is not a singleton case.```.

+ GCC confliction

  To avoid GCCcore confliction between python venv and aims.

  I load the python module first and then the aims denpendencies.
  ```
  module purge

  module load Python/3.10.4-GCCcore-11.3.0
  . /shared/home1/$USER/quip/bin/activate
  
  module load intel-compilers/2025.1.1 impi/2021.15.0-intel-compilers-2025.1.1 imkl/2024.2.0
  ```
  
  These are the modules with version change after this.
  >The following have been reloaded with a version change:
  >1) GCCcore/11.3.0 => GCCcore/14.2.0
  >2) binutils/2.38-GCCcore-11.3.0 => binutils/2.42-GCCcore-14.2.0
  >3) zlib/1.2.12-GCCcore-11.3.0 => zlib/1.3.1-GCCcore-14.2.0

  It complains, but the code runs as normal.
  >Lmod Warning:
  >
  >The following dependent module(s) are not currently loaded: GCCcore/11.3.0
  >
  >(required by: bzip2/1.0.8-GCCcore-11.3.0, ncurses/6.3-GCCcore-11.3.0,
  >libreadline/8.1.2-GCCcore-11.3.0, Tcl/8.6.12-GCCcore-11.3.0,
  >SQLite/3.38.3-GCCcore-11.3.0, XZ/5.2.5-GCCcore-11.3.0,
  >GMP/6.2.1-GCCcore-11.3.0, libffi/3.4.2-GCCcore-11.3.0,
  >Python/3.10.4-GCCcore-11.3.0), binutils/2.38-GCCcore-11.3.0 (required by:
  >Python/3.10.4-GCCcore-11.3.0), zlib/1.2.12-GCCcore-11.3.0 (required by:
  >Tcl/8.6.12-GCCcore-11.3.0, Python/3.10.4-GCCcore-11.3.0)

## Performance comparison with Hawk

CH3CO on a 4-by-4 7-layer Cu (100) slab.

+ 192-core calculation
  
  HAWK: 72 SCF cycles; 916 s; 13s/iteration; group compiled version **231208**--INTEL AVX512?; -5078049.234 eV
  
  FALCON: 109 SCF cycles; 1963 s; 18s/iteration; self-compiled version **250918**; -5078134.282 eV

  From aims **250822**,
  Using PBE minimal basis for metaGGAs by default (Sebastian Kokott, breaking change, can be overridden with atomic_solver_xc pw-lda)

+ 2-node calulation on Hawk
  72 SCF cycles; 1785s; 25s/iteration; group compiled version **231208**--INTEL AVX512?




  
