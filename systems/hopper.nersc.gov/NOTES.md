hopper.nersc.gov
================

For the GNU compiler, try the following compiler and linker flags:

    compiler = cc
    linker = CC
    extra_link_args =

When building Python it was necessary to load the `db` module.
See the `Modules-Setup` file in this directory for a working
`Modules/Setup` configuration when building Python.
