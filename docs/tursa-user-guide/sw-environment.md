# Software environment

The software environment on Tursa is primarily controlled through the
`module` command. By loading and switching software modules you control
which software and versions are available to you.

!!! information
    A module is a self-contained description of a software package -- it
    contains the settings required to run a software package and, usually,
    encodes required dependencies on other software packages.

By default, all users on Tursa start with the default software
environment loaded.

Software modules on Tursa are provided by both Eviden and by EPCC.

In this section, we provide:

   - A brief overview of the `module` command
   - A brief description of how the `module` command manipulates your
     environment

## Using the `module` command

We only cover basic usage of the `module` command here. For full
documentation please see the [Linux manual page on
modules](http://linux.die.net/man/1/module)

The `module` command takes a subcommand to indicate what operation you
wish to perform. Common subcommands are:

   - `module list [name]` - List modules currently loaded in your
     environment, optionally filtered by `[name]`
   - `module avail [name]` - List modules available, optionally
     filtered by `[name]`
   - `module savelist` - List module collections available (usually
     used for accessing different programming environments)
   - `module restore name` - Restore the module collection called
     `name` (usually used for setting up a programming environment)
   - `module load name` - Load the module called `name` into your
     environment
   - `module remove name` - Remove the module called `name` from your
     environment
   - `module swap old new` - Swap module `new` for module `old` in your
     environment
   - `module help name` - Show help information on module `name`
   - `module show name` - List what module `name` actually does to your
     environment

These are described in more detail below.

### Information on the available modules

The `module list` command will give the names of the modules and their
versions you have presently loaded in your environment. By default, you
will have no modules loaded when you first log into Tursa

Finding out which software modules are available on the system is
performed using the `module avail` command. To list all software modules
available, use:

```
[dc-user1@tursa-login1 ~]$ module avail
------------------------------------------------------ /home/y07/shared/tursa-modules -----------------------------------------------
cmake/3.27.4  nvhpc/23.5-nompi  setup-env  

----------------------------------------- /mnt/lustre/tursafs1/apps/cuda-12.3-modulefiles -------------------------------------------
cuda/12.3  openmpi/4.1.5-cuda12.3  ucx/1.15.0-cuda12.3  

------------------------------------------- /mnt/lustre/tursafs1/apps/cuda-11.4.1-modulefiles ---------------------------------------
cuda/11.4.1  openmpi/4.1.1-cuda11.4.1  ucx/1.12.0-cuda11.4.1  

------------------------------------------------- /mnt/lustre/tursafs1/apps/modulefiles --------------------------------
cuda/11.0.3  dot  gcc/9.3.0  module-git  module-info  modules  null  openmpi/4.0.4  openmpi/4.1.1  ucx/1.10.1  use.own  xpmem/2.6.5  
```

This will list all the names and versions of the modules available on
the service. Not all of them may work in your account though due to, for
example, licencing restrictions. You will notice that for many modules
we have more than one version, each of which is identified by a version
number. One of these versions is the default. As the service develops
the default version will change and old versions of software may be
deleted.

You can list all the modules of a particular type by providing an
argument to the `module avail` command. For example, to list all
available versions of the OpenMPI library, use:

```
[dc-user1@tursa-login1 ~]$ module avail openmpi
------------------------------------------------- /mnt/lustre/tursafs1/apps/cuda-12.3-modulefiles -------------------------------------------------
openmpi/4.1.5-cuda12.3  

------------------------------------------------ /mnt/lustre/tursafs1/apps/cuda-11.4.1-modulefiles ------------------------------------------------
openmpi/4.1.1-cuda11.4.1  

------------------------------------------------------ /mnt/lustre/tursafs1/apps/modulefiles ------------------------------------------------------
openmpi/4.0.4  openmpi/4.1.1 
```

The `module show` command reveals what operations the module actually
performs to change your environment when it is loaded. We provide a
brief overview of what the significance of these different settings mean
below. For example, for the default openmpi module:

```
[dc-user1@tursa-login1 ~]$ module show openmpi
-------------------------------------------------------------------
/mnt/lustre/tursafs1/apps/cuda-12.3-modulefiles/openmpi/4.1.5-cuda12.3:

module-whatis   Sets up OpenMPI on your environment
setenv          MPI_ROOT        /mnt/lustre/tursafs1/apps/basestack/cuda-12.3/openmpi/4.1.5-cuda12.3-slurm
prepend-path    PATH /mnt/lustre/tursafs1/apps/basestack/cuda-12.3/openmpi/4.1.5-cuda12.3-slurm/bin
prepend-path    LD_LIBRARY_PATH /mnt/lustre/tursafs1/apps/basestack/cuda-12.3/openmpi/4.1.5-cuda12.3-slurm/lib
prepend-path    MANPATH /mnt/lustre/tursafs1/apps/basestack/cuda-12.3/openmpi/4.1.5-cuda12.3-slurm/share/man
module load     ucx/1.15.0-cuda12.3
setenv          OMPI_CC cc
setenv          OMPI_CXX        g++
setenv          OMPI_CFLAGS     -g -m64
setenv          OMPI_CXXFLAGS   -g -m64
setenv          OMPI_MCA_pml    ucx
setenv          OMPI_MCA_osc    ucx
setenv          OMPI_MCA_btl    ^openib
-------------------------------------------------------------------
```

### Loading, removing and swapping modules

To load a module to use the `module load` command. For example, to load
the default version of OpenMPI into your environment, use:

```
[dc-user1@tursa-login1 ~]$ module load openmpi

        UCX 1.15.0 compiled with cuda 12.3 loaded

        OpenMPI 4.1.5 with cuda-12.3 and UCX 1.15.0 loaded

```

Once you have done this, your environment will be setup to use the OpenMPI library.
The above command will load the default version of
OpenMPI. If you need a specific version of the software, you can
add more information:

```
[dc-user1@tursa-login1 ~]$ module load openmpi/4.1.1-cuda11.4.1

        UCX 1.12.0 compiled with cuda 11.4.1 loaded


        OpenMPI 4.1.1 with cuda-11.4.1 and UCX 1.12.0  loaded

```

will load OpenMPI version 4.1.1 with CUDA 11.4.1 into your environment,
regardless of the default.

If you want to remove software from your environment, `module rm`
will remove a loaded module:

```
[dc-user1@tursa-login1 ~]$ module rm openmpi
```

will unload what ever version of `openmpi` (even if it is not the
default) you might have loaded.

There are many situations in which you might want to change the
presently loaded version to a different one, such as trying the latest
version which is not yet the default or using a legacy version to keep
compatibility with old data. This can be achieved most easily by using
`module swap oldmodule newmodule`.

Suppose you have loaded version 4.1.1 of `openmpi`, the following
command will change to version 4.1.1-cuda11.4.1:

```
[dc-user1@tursa-login1 ~]$ module swap openmpi openmpi/4.1.1-cuda11.4.1

        UCX 1.12.0 compiled with cuda 11.4.1 loaded


        OpenMPI 4.1.1 with cuda-11.4.1 and UCX 1.12.0  loaded

```

You did not need to specify the version of the loaded module in your
current environment as this can be inferred as it will be the only one
you have loaded.

### Capturing your environment for reuse

Sometimes it is useful to save the module environment that you are using
to compile a piece of code or execute a piece of software. This is saved
as a module collection. You can save a collection from your current
environment by executing:

```
[dc-user1@tursa-login1 ~]$ module save [collection_name]
```

!!! note
    If you do not specify the environment name, it is called `default`.

You can find the list of saved module environments by executing:

```
[dc-user1@tursa-login1 ~]$ module savelist
Named collection list:
 1) default
```

To list the modules in a collection, you can execute, e.g.,:

```
[dc-user1@tursa-login1 ~]$ module saveshow default
-------------------------------------------------------------------
/home/t01/t01/dc-user1/.module/default:

module use --append /mnt/lustre/tursafs1/apps/cuda-11.0.2-modulefiles
module use --append /mnt/lustre/tursafs1/apps/cuda-11.4.1-modulefiles
module use --append /mnt/lustre/tursafs1/apps/modulefiles
module load ucx/1.12.0-cuda11.4.1
module load openmpi/4.1.1-cuda11.4.1

-------------------------------------------------------------------
```

Note again that the details of the collection have been saved to the
home directory (the first line of output above). It is possible to save
a module collection with a fully qualified path, e.g.,

```
[dc-user1@tursa-login1 ~]$ module save /home/t01/t01/auser/my-module-collection
```

if you want to save to a specific file name.

To delete a module environment, you can execute:

```
[dc-user1@tursa-login1 ~]$ module saverm <environment_name>
```

### Restoring original environment: `module purge` plus reload

Unlike some other HPC systems, you cannot restore the original module environment
with just the `module purge` command. You also need to reload the base environment
setup module, i.e.

```bash
module purge
module load /home/y07/shared/tursa-modules/setup-env
```

## Shell environment overview

When you log in to Tursa, you are using the *bash* shell by default.
As any other software, the *bash* shell has loaded a set of environment
variables that can be listed by executing `printenv` or `export`.

The environment variables listed before are useful to define the
behaviour of the software you run. For instance, `OMP_NUM_THREADS`
define the number of threads.

To define an environment variable, you need to execute:

```
export OMP_NUM_THREADS=4
```

Please note there are no blanks between the variable name, the
assignation symbol, and the value. If the value is a string, enclose the
string in double quotation marks.

You can show the value of a specific environment variable if you print
it:

```
echo $OMP_NUM_THREADS
```

Do not forget the dollar symbol. To remove an environment variable, just
execute:

```
unset OMP_NUM_THREADS
```

## Compiler environment

The system supports two different primary compiler environments for GPU nodes:

* GCC GPU toolchain: GCC, CUDA 12.3, OpenMPI 4.1.5
* NVHPC GPU toolchain: NVHPC 23.5 (CUDA 12.1), OpenMPI 4.1.5

and one compiler environment for CPU nodes:

* GCC CPU toolchain: GCC, OpenMPI 4.1.5

### GCC GPU toolchain

To compile on the system for GPU nodes using the GCC toolchain, you would typically load the required modules:

```
module load gcc/9.3.0
module load cuda/12.3 
module load openmpi/4.1.5-cuda12.3


module list
Currently Loaded Modulefiles:
 1) /home/y07/shared/tursa-modules/setup-env   3) cuda/12.3             5) openmpi/4.1.5-cuda12.3  
 2) gcc/9.3.0                                  4) ucx/1.15.0-cuda12.3  
```

Once you have loaded the modules, the standard OpenMPI compiler wrapper
scripts are available:

- `mpicc`
- `mpicxx`
- `mpif90`

You can find more information on these scripts in the
[OpenMPI documentation](https://www.open-mpi.org/doc/v4.1/).

### NVHPC GPU toolchain

To compile on the system for GPU nodes using the GCC toolchain, you would typically load the required modules:

```
module load gcc/9.3.0
module load nvhpc/23.5-nompi
module load openmpi/4.1.5-cuda12.3
module list

Currently Loaded Modulefiles:
 1) /home/y07/shared/tursa-modules/setup-env   3) nvhpc/23.5-nompi      5) openmpi/4.1.5-cuda12.3  
 2) gcc/9.3.0                                  4) ucx/1.15.0-cuda12.3  
```

Once you have loaded the modules, the standard OpenMPI compiler wrapper
scripts are available:

- `mpicc`
- `mpicxx`
- `mpif90`

and the NVIDIA compilers are available as:

- `nvcc`
- `nvc++`
- `nvfortran`

!!! tip
    Both the NVIDIA compilers and the MPI compiler wrapper scripts will use the GCC
    compilers directly in the default configuration - this is often what you want. If
    you want the compiler wrappers to call the NVIDIA compilers themselves rather than 
    GCC directly, you would use:

    ```
    export OMPI_CC=nvcc
    export OMPI_CXX=nvc++
    export OMPI_FC=nvfortran
    ```

### GCC CPU toolchain

To compile on the system for CPU nodes using the GCC toolchain, you would typically load the required modules:

```
module load gcc/9.3.0
module load openmpi/4.1.5


module list
Currently Loaded Modulefiles:
 1) /home/y07/shared/tursa-modules/setup-env   3) openmpi/4.1.5 
 2) gcc/9.3.0                                  4) ucx/1.15.0  
```

Once you have loaded the modules, the standard OpenMPI compiler wrapper
scripts are available:

- `mpicc`
- `mpicxx`
- `mpif90`

You can find more information on these scripts in the
[OpenMPI documentation](https://www.open-mpi.org/doc/v4.1/).


## Other build tools

### cmake

CMake is available by using the commands:

```
[dc-user1@tursa-login1 ~]$ module load /home/y07/shared/tursa-modules/setup-env
[dc-user1@tursa-login1 ~]$ module load cmake
```
