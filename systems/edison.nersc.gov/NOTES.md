edison.nersc.gov
================

For the Intel compiler, try the following compiler and linker flags:

    compiler = cc
    linker = ftn -nofor_main -cxxlib
    extra_link_args = -mkl=cluster -lmpichcxx_intel

When building Python it was necessary to load the `bzip2` module.
See the `Modules-Setup` file in this directory for a working
`Modules/Setup` configuration when building Python.

To compile C++11 when using the Intel compiler, it may be necessary
to load the `gcc` module.
