# Spack

[Spack](https://github.com/spack/spack) is a package manager, a tool to assist
with building and installing software as well as determining what dependencies
are required and installing those. It was originally designed for use on HPC
clusters, where several variations of a given package may be installed alongside
one another for different use cases -- for example different versions, built
with different compilers, using MPI or hybrid MPI+OpenMP. Spack is principally
written in Python but has a component written in Answer Set Programming (ASP)
which is used to determine the required dependencies for a given package
installation.

Users are welcome to install Spack themselves in their own directories, but we
are making an experimental installation tailored for Tursa available
centrally. This page provides documentation on how to activate and install
packages using the central installation on Tursa. For more in-depth
information on using Spack itself please see the [developers'
documentation](https://spack.readthedocs.io/en/latest/).

!!! important
    As the Tursa central Spack installation is still in an experimental stage
    please be aware that we cannot guarantee that it will work with full
    functionality and we may not be able to provide support.

## Activating Spack

You activate Spack in your environment on Tursa with the command:

```
source /home/y07/shared/utils/core/spack/share/spack/setup-env.sh
```

This configures Spack to place its cache on and install software to a directory
called `.spack` in your base home directory, e.g. at
`/home/dp001/dp001/dc-user1/.spack`.

At this point Spack is available to you via the `spack` command. You can get
started with `spack help`, reading the [Spack
documentation](https://spack.readthedocs.io/en/latest/), or by testing a
package's installation.

## Using Spack on Tursa

### Installing software

At its simplest, Spack installs software with the `spack install` command:

    [dc-user1@tursa-login1 ~]$ spack install gromacs

This very simple `gromacs` installation specification, or spec, would install
GROMACS using the default options given by the Spack `gromacs` package. The spec
can be expanded to include which options you like. For example, the command

    [dc-user1@tursa-login1 ~]$ spack install gromacs@2024.3 %gcc +mpi +cuda

would use the GCC compiler to install an MPI-enabled and CUDA-enabled version of GROMACS version
2024.3.

!!! tip
    Spack needs to bootstrap the installation of some extra software in order to
    function, principally `clingo` which is used to solve the dependencies
    required for an installation. The first time you ask Spack to concretise a
    spec into a precise set of requirements, it will take extra time as it
    downloads this software and extracts it into a local directory for Spack's
    use.

You can find information about any Spack package and the options
available to use with the `spack info` command:

    [dc-user1@tursa-login1 ~]$ spack info gromacs

!!! tip
    The Spack developers also provide a website at
    [https://packages.spack.io/](https://packages.spack.io/) where you can
    search for and examine packages, including all information on options,
    versions and dependencies.

When installing a package, Spack will determine what dependencies are required
to support it. If they are not already available to Spack, either as packages
that it has installed beforehand or as external dependencies, then Spack will
also install those, marking them as implicitly installed, as opposed to the
explicit installation of the package you requested. If you want to see the
dependencies of a package before you install it, you can use `spack spec` to see
the full concretised set of packages:

    [dc-user1@tursa-login1 ~]$ spack spec gromacs@2024.2 %gcc +mpi +cuda

!!! tip
    Spack on Tursa has been configured to use already installed versions of
    some software, e.g., OpenMPI, CUDA.

### Using Spack packages

Spack provides a module-like way of making software that you have installed
available to use. If you have a GROMACS installation, you can make it
available to use with `spack load`:

    [dc-user1@tursa-login1 ~]$ spack load gromacs

At this point you should be able to use the software as normal. You can then
remove it once again from the environment with `spack unload`:

    [dc-user1@tursa-login1 ~]$ spack unload gromacs

If you have multiple variants of the same package installed, you can use the
spec to distinguish between them. You can always check what packages have been
installed using the `spack find` command. If no other arguments are given it
will simply list all installed packages, or you can give a package name to
narrow it down:

    [dc-user1@tursa-login1 ~]$ spack find gromacs

You can see your packages' install locations using `spack find --paths` or
`spack find -p`.

### Maintaining your Spack installations

In any Spack command that requires as an argument a reference to an installed
package, you can provide a hash reference to it rather than its spec. You can
see the first part of the hash by running `spack find -l`, or the full hash with
`spack find -L`. Then use the hash in a command by prefixing it with a forward
slash, e.g. `wjy5dus` becomes `/wjy5dus`.

If you have two packages installed which appear identical in `spack find` apart
from their hash, you can differentiate them with `spack diff`:

    [dc-user1@tursa-login1 ~]$ spack diff /wjy5dus /bleelvs

You can uninstall your packages with `spack uninstall`:

    [dc-user1@tursa-login1 ~]$ spack uninstall gromacs@2024.2

and of course, to be absolutely certain that you are uninstalling the correct
package, you can provide the hash:

    [dc-user1@tursa-login1 ~]$ spack uninstall /wjy5dus

Uninstalling a package will leave behind any implicitly installed packages that
were installed to support it. Spack may have also installed build-time
dependencies that aren't actually needed any more -- these are often packages
like `autoconf`, `cmake` and `m4`. You can run the garbage collection command to
uninstall any build dependencies and implicit dependencies that are no longer
required:

    [dc-user1@tursa-login1 ~]$ spack gc

If you commonly use a set of Spack packages together you may want to consider
using a Spack environment to assist you in their installation and management.
Please see the [Spack
documentation](https://spack.readthedocs.io/en/latest/environments.html) for
more information.

## Custom configuration

Spack is configured using YAML files. The central installation on Tursa made
available to users is configured to use pre-installed software and
to allow you to start installing software to your `/home` directories right
away, but if you wish to make any changes you can provide your own overriding
userspace configuration.

Your own configuration should fit in the user level scope. On Tursa Spack is
configured to, by default, place and look for your configuration files in your
home directory at e.g. `/home/dp001/dp001/dc-user1/.spack`. You can however override
this to have Spack use any directory you choose by setting the
`SPACK_USER_CONFIG_PATH` environment variable, for example:

    [dc-user1@tursa-login1 ~]$ export SPACK_USER_CONFIG_PATH=/home/dp001/dp001/dc-user1/spack-config

Of course this will need to be a directory where you have write permissions,
such in your home or work directories, or in one of your project's `shared`
directories.

You can edit the configuration files directly in a text editor or by
running, for example:

    [dc-user1@tursa-login1 ~]$ spack config edit repos

which would open your `repos.yaml` in `vim`.

!!! tip
    If you would rather not use `vim`, you can change which editor is used by
    Spack by setting the `SPACK_EDITOR` environment variable.

The final configuration used by Spack is a compound of several scopes, from the
Spack defaults which are overridden by the Tursa system configuration files,
which can then be overridden in turn by your own configurations. You can see
what options are in use at any point by running, for example:

    [dc-user1@tursa-login1 ~]$ spack config get config

which goes through any and all `config.yaml` files known to Spack and sets
the options according to those files' level of precedence. You can also get more
information on which files are responsible for which lines in the final active
configuration by running, for example to check `packages.yaml`:

    [dc-user1@tursa-login1 ~]$ spack config blame packages

Unless you have already written a `packages.yaml` of your own, this will show a
mix of options originating from the Spack defaults and also from the configuration
we have created on Tursa.

If there is some behaviour in Spack that you want to change, looking at the
output of `spack config get` and `spack config blame` may help to show what you
would need to do. You can then write your own user scope configuration file to
set the behaviour you want, which will override the option as set by the
lower-level scopes.

Please see the [Spack
documentation](https://spack.readthedocs.io/en/latest/configuration.html) to
find out more about writing configuration files.

## Writing new packages

A Spack package is at its core a Python `package.py` file which provides
instructions to Spack on how to obtain source code and compile it. A very simple
package will allow it to build just one version with one compiler and one set of
options. A more fully-featured package will list more versions and include logic
to build them with different compilers and different options, and to also pick
its dependencies correctly according to what is chosen.

Spack provides several thousand packages in its `builtin` repository. You may be
able to use these with no issues on Tursa by running `spack install` as
described above, but if you do run into problems then you may wish to write your own.

### Creating your own package repository

A package repository is a directory containing a `repo.yaml` configuration file
and another directory called `packages`. Directories within the latter are named
for the package they provide, for example `cp2k`, and contain in turn a
`package.py`. You can create a repository from scratch with the command

    [dc-user1@tursa-login1 ~]$ spack repo create dirname

where `dirname` is the name of the directory holding the repository. This
command will create the directory in your current working directory, but you can
choose to instead provide a path to its location. You can then make the new
repository available to Spack by running:

    [dc-user1@tursa-login1 ~]$ spack repo add dirname

This adds the path to `dirname` to the `repos.yaml` file in your user scope
configuration directory [as described above](#custom-configuration). If your
`repos.yaml` doesn't yet exist, it will be created.

A Spack repository can similarly be removed from the config using:

    [dc-user1@tursa-login1 ~]$ spack repo rm dirname

### Namespaces and repository priority

A package can exist in several repositories. To distinguish between these packages,
each repository's packages exist within that repository's namespace. By default
the namespace is the same as the name of the directory it was created in, but Spack
does allow it to be different. 

!!! tip
    If you want your repository namespace to be different from the name of
    the directory, you can change it either by editing the repository's
    `repo.yaml` or by providing an extra argument to `spack repo create`:

        [dc-user1@tursa-login1 ~]$ spack repo create dirname namespace

Running `spack find -N` will return the list of installed packages with their
namespace. You'll see that they are then prefixed with the repository namespace,
for example `builtin.bison@3.8.2` and `tursa.quantum-espresso@7.2`. In order
to avoid ambiguity when managing package installation you can always prefix a
spec with a repository namespace.

If you don't include the repository in a spec, Spack will search in order all the
repositories it has been configured to use until it finds a matching
package, which it will then use. The earlier in the list of repositories, the
higher the priority. You can check this with:

    [dc-user1@tursa-login1 ~]$ spack repo list

If you run this without having added any repositories of your own, you will see
that the `builtin` repository is the only one available.

### Creating a package

Once you have a repository of your own in place, you can create new packages to
store within it. Spack has a `spack create` command which will do the initial
setup and create a boilerplate `package.py`. To create an empty package called
`packagename` you would run:

    [dc-user1@tursa-login1 ~]$ spack create --name packagename

However, it will very often be more efficient if you instead provide a download
URL for your software as the argument. For example, the Code_Saturne 8.0.3
source is obtained from
`https://www.code-saturne.org/releases/code_saturne-8.0.3.tar.gz`,
so you can run:

    [dc-user1@tursa-login1 ~]$ spack create https://www.code-saturne.org/releases/code_saturne-8.0.3.tar.gz

Spack will determine from this the package name, the download URLs for all
versions X.Y.Z matching the
`https://www.code-saturne.org/releases/code_saturne-X.Y.Z.tar.gz` pattern. It
will then ask you interactively which of these you want to use. Finally, it will
download the `.tar.gz` archives for those versions and calculate their
checksums, then place all this information in the initial version of the package
for you. This takes away a lot of the initial work!

At this point you can get to work on the package. You can edit an existing
package by running

    [dc-user1@tursa-login1 ~]$ spack edit packagename

or by directly opening `packagename/package.py` within the repository with a
text editor.

The boilerplate code will note several sections for you to fill out. If you did
provide a source code download URL, you'll also see listed the versions you
chose and their checksums.

A package is implemented as a Python class. You'll see that by default it will
inherit from the `AutotoolsPackage` class which defines how a package following
the common `configure` > `make` > `make install` process should be built. You
can change this to another build system, for example `CMakePackage`. If you
want, you can have the class inherit from several different types of build
system classes and choose between them at install time.

Options must be provided to the build. For an `AutotoolsPackage` package, you
can write a `configure_args` method which very simply returns a list of the
command line arguments you would give to `configure` if you were building the
code yourself. There is an identical `cmake_args` method for `CMakePackage`
packages.

Finally, you will need to provide your package's dependencies. In the main body
of your package class you should add calls to the `depends_on()` function. For
example, if your package needs MPI, add `depends_on("mpi")`. As the argument to
the function is a full Spack spec, you can provide any necessary versioning or
options, so, for example, if you need PETSc 3.18.0 or newer with Fortran support,
you can call `depends_on("petsc+fortran@3.18.0:")`.

If you know that you will only ever want to build a package one way, then
providing the build options and dependencies should be all that you need to do.
However, if you want to allow for different options as part of the install spec,
patch the source code or perform post-install fixes, or take more manual control
of the build process, it can become much more complex. Thankfully the Spack
developers have provided excellent documentation covering the whole process, and
there are many existing packages you can look at to see how it is done.

