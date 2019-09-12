**This page is under construction!**

# Existing users

If you are a user of [Spack](https://github.com/spack/spack) you might be glad to hear that preCICE and [ openFOAM
adapter for preCICE ](https://github.com/precice/openfoam-adapter) made their way in the Spack repository, starting
from this  [commit](https://github.com/spack/spack/commit/f946a83c8cb929dd0021d8a5a2a0a1d99ba2e47d)

Therefore they can be readily used e.g:

```bash
spack install precice ^boost@1.65.1
```
to fetch the latest development version of preCICE.
And for the openFOAM adapter:
```bash
spack install of-precice
```

# New users

In case if you have not used Spack before, you might want to give it a try. Here is a good [start](
https://spack.readthedocs.io/en/latest/getting_started.html)

It is a
> multi-platform package manager that builds and installs multiple versions and configurations of software. It works on Linux, macOS, and many supercomputers.
  Spack is non-destructive: installing a new version of a package does not break existing installations, so many configurations of the same package can coexist.

After installing a package, you can use it with `spack load <package>` (requires e.g. [Environment Modules](http://modules.sourceforge.net/) to be installed). Read more in the [Spack documentation](https://spack.readthedocs.io/en/latest/module_file_support.html).


When you try to install a package for the first time, quite a lot of dependency packages would be installed, even if they are already present on your system. You can view this list by:

```bash
  spack spec precice
```

You can specify dependencies, that are already installed on your system by modifying your preferences in the
`~/.spack/packages.yaml`.

E.g. to use your own version of MPI

```yaml
packages:
    openmpi:
        paths:
            openmpi@1.10.1: /opt/local
        buildable: False
```

Additionally you might want to opt out some installation options for dependencies of precice such as Eigen and Boost.  
You can browse options with `spack info boost` and `spack info eigen`.  For instance to browse dependencies of only essential boost libraries,  that are used by preCICE you need to disable some of the default options:

```bash
./bin/spack install precice ^boost@1.65.1  -atomic -chrono -date_time -exception -graph -iostreams -locale -math -random -regex -serialization -signals -timer -wave ^eigen@3.3.1 -fftw -metis -mpfr -scotch -suitesparse ^openmpi@3.1.2
```

Similarly for minimal Eigen version:

```bash
./bin/spack spec eigen@3.3.1 -fftw -metis -mpfr -scotch -suitesparse
```

Overall, then the command for the minimal installation of preCICE could look like

```bash

./bin/spack install precice ^boost@1.65.1  -atomic -chrono -date_time -exception -graph -iostreams -locale -math -random -regex -serialization -signals -timer -wave ^eigen@3.3.1 -fftw -metis -mpfr -scotch -suitesparse ^openmpi@3.1.2
```

After some installation time, preCICE will be located in the folder in `$SPACK_FOLDER/opt/spack` based on the compiler and the system. To find out the directory, where preCICE is installed run the following
```bash
spack find -p precice
```
This can give such output
```bash
opt/spack/linux-ubuntu18.04-x86_64/gcc-7.3.0/precice-working-p5f6hf5u3dmyeepa6jwg4j4xyalbn4uc/bin/
```

You can also view packages that are installed with  

```bash
spack find  
```

That can have such output:
```bash
autoconf@2.69    boost@1.60.0  boost@1.67.0  cmake@3.12.3   eigen@3.3.1  hwloc@1.11.9         libsigsegv@2.11  libxml2@2.9.8  ncurses@6.1     openmpi@3.1.2   perl@5.26.2    precice@working  util-macros@1.19.1  zlib@1.2.11
automake@1.16.1  boost@1.60.0  bzip2@1.0.6   diffutils@3.6  gdbm@1.14.1  libpciaccess@0.13.5  libtool@2.4.6    m4@1.4.18      numactl@2.0.11  openssl@1.0.2o  pkgconf@1.4.2  readline@7.0     xz@5.2.4
```

To unistall a package simply run:
```bash
spack unistall precice

```
One we have preCICE installed, we can also install openFOAM-adapter:

```bash  
spack install of-precice
```

If we don't want to build openFOAM from source, but rather use the one already present in the system (or install with the package manager) we need to add configuration in the `~/.spack/packages.yaml` as follows:
```yml
packages:
  openfoam-org:
      paths:
        openfoam-org@5.0: /opt/openfoam5
      buildable: False
```

Openfoam is a [virtual dependency](https://spack.readthedocs.io/en/latest/basic_usage.html#virtual-dependencies)  (meaning it can satisfied both by openfoam-com and openfoam-org), so we want to select the provider for it.
```
spack install openfoam ^openfoam-org
```

In order to install openFOAM adapter, we need to have to specify on just installed
preCICE. We can either write the same specification as before (with selected boost libraries and Eigen versions and options ), or refer to specification by its hash:

```bash
spack find --long precice
```
```bash
-- linux-ubuntu18.04-x86_64 / gcc@7.3.0 -------------------------
dlukq7q precice@develop
```

Now, we can install openfoam-adapter with:
```bash
spack install of-precice ^openfoam@5.0 ^/dlukq7q
```

# Troubleshooting

## Mixing up dependencies from system and Spack

In case you see that Spack reports an error in which points to a non-spack directory, maybe you are forcing your compiler to look at places it shouldn't. Please avoid setting environment variables such as `CPLUS_INCLUDE_PATH`, `LD_LIBRARY_PATH`, or `LIBRARY_PATH` when using Spack. This can particularly lead to problems if you have a (broken) installation of a requested dependency in that path.

Keep in mind that, by default, Spack will use the default compiler of your system.

## Boost-related errors

By default, Spack will use the latest version of Boost. As Boost changes frequently, it is possible to get an error either because of a change that we have not yet incorporated, or because your compiler is too old.

Typical example (GCC 5.4.0, Boost 1.70.0, preCICE 1.5.0):
```
error: 'stream_socket_service' in namespace 'boost::asio' does not name a template type
```

In most cases, you should be able to resolve this by downgrading your Boost version, e.g.:

```
spack install precice ^boost@1.68.0
```

## Freezing during calls to MPI

If you build preCICE with a different version/implementation of MPI than you used for your solver, you may encounter unexpected behavior early at runtime, e.g. during configuration. In that case, make sure to build preCICE with the same version, e.g.:

```
spack install precice ^openmpi@1.10.2
```

You could also use the MPI library installed in your system (see above).

## Cannot find build/last link

(update: this should be fixed by now and no workaround should be required)

It seems like the current development version of preCICE does not get built succesfully with Spack, returning an error message like:

```
CMake Error: failed to create symbolic link '[spack dir]/var/spack/stage/precice-master-[hash]/precice/build/last': no such file or directory
```

This is due to important changes we are introducing to CMake, trying to offer closer to the standard behavior. Until we fix this issue, you might install preCICE package from the working commit. Edit the file `[spack directory]/var/spack/repos/builtin/packages/precice/package.py` and add an alternative version:

```python  

version('develop', branch='develop')
# this line should be added
version('working', branch='develop', commit='f00c1866cfa950f7dcdf9c802d6f34fa52e98d98')

```

And then use `precice@working` instead of `precice` in the instructions above. For example:

```
./bin/spack install precice@working ^boost@1.65.1  -atomic -chrono -date_time -exception -graph -iostreams -locale -math -random -regex -serialization -signals -timer -wave ^eigen@3.3.1 -fftw -metis -mpfr -scotch -suitesparse ^openmpi@3.1.2
```

## Problem building dependencies

There might be conflicts between dependencies. If conflicts are detected, Spack prints an error message starting with `Error: Conflicts in concretized spec`. In this case you may leave out some dependencies (see above) in order to avoid conflicts.

**Example (on Ubuntu 18.04):**
The following error message was generated on Ubuntu 18.04 LTS using the command `spack spec precice` (Find a solution to the problem below):
```
Input spec
--------------------------------
precice

Concretized
--------------------------------
==> Error: Conflicts in concretized spec "precice@develop%gcc@7.3.0 build_type=RelWithDebInfo +mpi~petsc~python+shared arch=linux-ubuntu18.04-x86_64 /qhatpnl"

List of matching conflicts for spec:

    flex@2.6.4%gcc@7.3.0+lex arch=linux-ubuntu18.04-x86_64
        ^bison@3.0.5%gcc@7.3.0 arch=linux-ubuntu18.04-x86_64
            ^diffutils@3.6%gcc@7.3.0 arch=linux-ubuntu18.04-x86_64
            ^help2man@1.47.4%gcc@7.3.0 arch=linux-ubuntu18.04-x86_64
                ^gettext@0.19.8.1%gcc@7.3.0+bzip2+curses+git~libunistring+libxml2 patches=9acdb4e73f67c241b5ef32505c9ddf7cf6884ca8ea661692f21dca28483b04b8 +tar+xz arch=linux-ubuntu18.04-x86_64
                    ^bzip2@1.0.6%gcc@7.3.0+shared arch=linux-ubuntu18.04-x86_64
                    ^libxml2@2.9.8%gcc@7.3.0~python arch=linux-ubuntu18.04-x86_64
                        ^pkgconf@1.4.2%gcc@7.3.0 arch=linux-ubuntu18.04-x86_64
                        ^xz@5.2.4%gcc@7.3.0 arch=linux-ubuntu18.04-x86_64
                        ^zlib@1.2.11%gcc@7.3.0+optimize+pic+shared arch=linux-ubuntu18.04-x86_64
                    ^ncurses@6.1%gcc@7.3.0~symlinks~termlib arch=linux-ubuntu18.04-x86_64
                    ^tar@1.30%gcc@7.3.0 arch=linux-ubuntu18.04-x86_64
                ^perl@5.26.2%gcc@7.3.0+cpanm patches=0eac10ed90aeb0459ad8851f88081d439a4e41978e586ec743069e8b059370ac +shared+threads arch=linux-ubuntu18.04-x86_64
                    ^gdbm@1.14.1%gcc@7.3.0 arch=linux-ubuntu18.04-x86_64
                        ^readline@7.0%gcc@7.3.0 arch=linux-ubuntu18.04-x86_64
            ^m4@1.4.18%gcc@7.3.0 patches=3877ab548f88597ab2327a2230ee048d2d07ace1062efe81fc92e91b7f39cd00,c0a408fbffb7255fcc75e26bd8edab116fc81d216bfd18b473668b7739a4158e,fc9b61654a3ba1a8d6cd78ce087e7c96366c290bc8d2c299f09828d793b853c8 +sigsegv arch=linux-ubuntu18.04-x86_64
                ^libsigsegv@2.11%gcc@7.3.0 arch=linux-ubuntu18.04-x86_64

1. "%gcc@7.2.0:" conflicts with "flex@2.6.4"
```
**Solution**: It was possible to install preCICE using only the essential boost libraries (see above).

## Module: Command not found

Spack works great with [Environment Modules](http://modules.sourceforge.net/), but it needs to be installed. Check [this page](https://spack.readthedocs.io/en/latest/module_file_support.html) and [this answer for Ubuntu](https://askubuntu.com/a/533636/142834).

## How to run the unit tests?

The script `compileAndTest.py`, as well as the binary `testprecice` are currently not provided with the package.
