Slither: Static Python builds for HPC Systems
=============================================

Slither is a set of patches for Python (and related modules) and a
command line tool for building static CPython binaries. In addition
slither supports byte-compiling Python module sources into
[frozen modules](http://docs.python.org/2/c-api/import.html#PyImport_FrozenModules).

In the most optimized configuration, Slither is capable of producing
binaries which are entirely self-contained and strictly minimize the
number of file system calls (i.e. `stat` and `open`).


Rationale
---------

While newer HPC systems do support dynamic libraries, the support
remains unoptimized. Locating and loading each shared object can
result in a lot of file system contention, especially for larger jobs
involving thousands of processors. This contention is also present in
importing regular Python (`.py`) modules. Depending on the Python
configuration, each imported module can require ten or more `stat`
system calls.

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

1. Build a custom Python 2.7.3 interpreter which contains a patched
version of distutils.

2. Use the custom interpreter to build and install Python
modules. Most modules can be built with little or no additional
configuration.

3. Use the `bin/slither` script to build custom, statically linked,
Python interpreters which bake in Python module byte-code.

Getting Started
---------------

Please see the `INSTALL.md` for installation instructions.


Alternative Approaches
----------------------

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
* [GPAW Cray XT4/XT5 Instructions](https://wiki.fysik.dtu.dk/gpaw/install/Cray/louhi.html)
* [GPAW Cray XT5 Instructions](https://wiki.fysik.dtu.dk/gpaw/install/Cray/jaguar.html)
* [pyprop Cray XT4 Instructions](https://code.google.com/p/pyprop/wiki/Installation_CrayXT4)
* [cx_Freeze](http://cx-freeze.sourceforge.net/)
* [MPI_Import](https://github.com/langton/MPI_Import)
* [walla](https://bitbucket.org/wscullin/walla)
* [collfs](https://github.com/ahmadia/collfs)
* [DLcache](https://www.nersc.gov/assets/userguide.txt)
* [Statically Linking Python](http://mdqinc.com/blog/2011/08/statically-linking-python-with-cython-generated-modules-and-packages/)
