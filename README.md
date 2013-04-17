Slither: Static Python builds for HPC Systems
=============================================

Slither is a set of patches for Python (and related modules) and a
command line tool for building static CPython binaries. In addition
slither supports byte-compiling Python module sources into
[frozen modules](http://docs.python.org/2/c-api/import.html#PyImport_FrozenModules).

In the most optimized configuration, Slither is capable of producing
binaries which are entirely self-contained and strictly minimize the
number of filesystem syscalls (i.e. `stat`s and `open`s).


Rationale
---------

While newer HPC systems do support dynamic libraries, the support
remains unoptimized. Locating and loading each shared object can
result in a lot of filesystem contention, especially for larger jobs
involving thousands of processors. This contention is also present in
importing regular Python (`.py`) modules. Depending on the Python
configuration, each imported module can require ten or more filesystem
`stat`s.

This contention manifests itself as a very long start up time --- the
amount of time required for the Python interpreter to start and the
prerequisite modules to be imported. On one node, this is often under
a minute and is generally ignorable.  On a hundred nodes this can
take hours. See
[Shared Library Performance on Hopper](https://cug.org/proceedings/attendee_program_cug2012/includes/files/pap124.pdf)
by Z. Zhao et al. and
[Python in a Parallel Environment](http://www.nersc.gov/assets/Uploads/GroteNUG2013.pdf) by D. Grote.


Previous Work
-------------

The [GPAW](https://wiki.fysik.dtu.dk/gpaw/index.html) package
[static python builds](https://wiki.fysik.dtu.dk/gpaw/install/Cray/jaguar.html)
provided instructions and patches for static Python builds, which this
project builds upon.


Method of Operation
-------------------

This project is composed of the following pieces.

1. A custom Python 2.7.3 build:

  * All Python standard library extensions are statically compiled
    into the executable.

  * A patched verison of distutils which generates `.a` archives
    instead of `.so` dynamic libraries.

2. A command line tool `slither` which reads a configuration file
   and compiles a custom Python intpreter.  Sample configuration
   files can be found in the `examples/` directory.


Getting Started
===============

Please see the `INSTALL.md` for installation instructions.


Alternative Approaches
======================

Some tools have been developed to speed up Python startup times. For
example, NERSC provides the DLcache and FMcache tools on Hopper.

DLcache is a general purpose tool to optimize the importing of shared
objects (i.e., `dlopen`). FMcache is a more specialized tool to
optimize the importing of Python modules (`.py` and `.pyc` files).  In
each case a cache is built by executing the program on a small number
of MPI ranks. The cache is then distributed and the job run on a large
number of MPI ranks. The main drawbacks are the following:

* It must be easy to run the same job (or at least the same beginning
  import statements) at small width. Often this means creating a
  separate `.py` script which just imports the modules you will be
  using.

* The libraries are loaded in the same order on every rank. In
  practice this means loading every module you might use at the
  beginning of your script.

* The batch job script must be altered with several pre- and
  post-processing steps.


Resources
---------

* [BuildStatically](http://wiki.python.org/moin/BuildStatically)
* [GPAW Cray XT5 Instructions](https://wiki.fysik.dtu.dk/gpaw/install/Cray/jaguar.html)
