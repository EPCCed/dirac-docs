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

Software modules on Tursa are provided by both ATOS and by EPCC.

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
versions you have presently loaded in your environment:

```
auser@login01:~> module list
Currently Loaded Modulefiles:
```

Finding out which software modules are available on the system is
performed using the `module avail` command. To list all software modules
available, use:

```
auser@login01:~> module avail
 
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
auser@login01:~> module avail openmpi


```

If you want more info on any of the modules, you can use the `module
help` command:

```
auser@login01:~> module help openmpi


```

The `module show` command reveals what operations the module actually
performs to change your environment when it is loaded. We provide a
brief overview of what the significance of these different settings mean
below. For example, for the default openmpi module:

```
auser@login01:~> module show openmpi
-
```

### Loading, removing and swapping modules

To load a module to use the `module load` command. For example, to load
the default version of OpenMPI into your environment, use:

```
auser@login01:~> module load openmpi
```

Once you have done this, your environment will be setup to use the OpenMPI library.
The above command will load the default version of
OpenMPI. If you need a specific version of the software, you can
add more information:

```
auser@login01:~> module load cray-fftw/4.0.5
```

will load OpenMPI version 4.0.5 into your environment,
regardless of the default.

If you want to remove software from your environment, `module remove`
will remove a loaded module:

```
auser@login01:~> module remove openmpi
```

will unload what ever version of `openmpi` (even if it is not the
default) you might have loaded.

There are many situations in which you might want to change the
presently loaded version to a different one, such as trying the latest
version which is not yet the default or using a legacy version to keep
compatibility with old data. This can be achieved most easily by using
`module swap oldmodule newmodule`.

Suppose you have loaded version 4.0.4 of `openmpi`, the following
command will change to version 4.0.5:

```
auser@login01:~> module swap openmpi openmpi/4.0.5
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
auser@login01:~> module save [collection_name]
```

!!! note
    If you do not specify the environment name, it is called `default`.

You can find the list of saved module environments by executing:

```
auser@login01:~> module savelist
Named collection list:
 1) default
```

To list the modules in a collection, you can execute, e.g.,:

```
auser@login01:~> module saveshow default
-------------------------------------------------------------------
/home/t01/t01/auser/.module/default:
module use --append /opt/cray/pe/perftools/20.09.0/modulefiles
module use --append /opt/cray/pe/craype/2.7.0/modulefiles
module use --append /usr/local/Modules/modulefiles
module use --append /opt/cray/pe/cpe-prgenv/7.0.0
module use --append /opt/modulefiles
module use --append /opt/cray/modulefiles
module use --append /opt/cray/pe/modulefiles
module use --append /opt/cray/pe/craype-targets/default/modulefiles
module load cpe-gnu
module load gcc
module load craype
module load craype-x86-rome
module load --notuasked libfabric
module load craype-network-ofi
module load cray-dsmml
module load perftools-base
module load xpmem
module load cray-mpich
module load cray-libsci
module load /work/y07/shared/tursa-modules/modulefiles-cse/epcc-setup-env
```

Note again that the details of the collection have been saved to the
home directory (the first line of output above). It is possible to save
a module collection with a fully qualified path, e.g.,

```
auser@login1:~> module save /work/t01/z01/auser/.module/myenv
```

which would make it available from the batch system.

To delete a module environment, you can execute:

```
auser@login01:~> module saverm <environment_name>
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
