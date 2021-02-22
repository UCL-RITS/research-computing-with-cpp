---
title: MPI in practice
---

## MPI in practice




### Specification and implementation

* In practice, we use MPI, the [Message Passing Interface](http://en.wikipedia.org/wiki/Message_Passing_Interface)
* MPI is a **specification** for a **library**
* It is implemented by separate vendors/open-source projects
     - [OpenMPI](http://www.open-mpi.org/)
     - [MPICH](http://www.mpich.org/)
* It is a C library with many many bindings:
     - Fortran (part of official MPI specification)
     - Python: [boost](http://www.boost.org/doc/libs/1_55_0/doc/html/mpi/python.html), [mpi4py](http://mpi4py.scipy.org/)
     - R: [Rmpi](http://cran.r-project.org/web/packages/Rmpi/index.html)
     - C++: [boost](http://www.boost.org/doc/libs/1_57_0/doc/html/mpi.html)

### Programming and running

* An MPI program is executed with ``mpiexec -n N [options] nameOfProgram [args]``

* MPI programs call methods from the mpi library

``` cpp
int MPI_Bcast(void *buffer, int count, MPI_Datatype datatype, int root,
               MPI_Comm comm)
```

* Vendors provide wrappers (mpiCC, mpic++) around compilers.
  Wrappers point to header file locations and link to the right libraries.
  An MPI program can be (easily) compiled by substituting ``(g++|icc) -> mpiCC``

### Hello, world!: hello.cc

{% code cpp/hello.cc %}

### Hello, world!: CMakeLists.txt

``` CMake
find_package(MPI REQUIRED)

add_executable(hello hello.cc)
target_include_directories(hello SYSTEM PUBLIC ${MPI_INCLUDE_DIRS})
target_link_libraries(hello PUBLIC ${MPI_LIBRARIES})
```

### Hello, world!: compiling and running

On `aristotle.rc.ucl.ac.uk`:

- Load modules:

``` bash
  module swap compilers/gnu
  module unload mpi/intel/2015/update3/intel
  module load mpi/openmpi/3.0.0/gnu-4.9.2
  module load cmake
```

- Create files "hello.cc" and "CMakeLists.txt" in some directory
- Create build directory: ``mkdir build && cd build``
- Run cmake and make: ``cmake .. && make``
- Run the code: ``mpiexec -n 4 hello``

### Hello, world! dissected

- MPI calls **must** appear beween ``MPI_Init`` and ``MPI_Finalize``
- Groups of processes are handled by a communicator. `MPI_COMM_WORLD` handles
    the group of all processes.
    ![]({% figurepath %}world.png)

- All know size of group and rank (order) of process in group
- By **convention**, process of rank 0 is **special** and called **root**

### MPI with CATCH

Running MPI unit-tests requires ``MPI_Init`` and ``MPI_Finalize`` before and after the
test framework (**not** inside the tests).

{% code cpp/helloCatch.cc %}
