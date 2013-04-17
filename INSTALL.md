Getting Started
===============

Building Python
---------------

1. Download and extract the CPython 2.7.3 source.

        wget http://python.org/ftp/python/2.7.3/Python-2.7.3.tar.bz2
        tar xf Python-2.7.3.tar.bz2
        cd Python-2.7.3

2. Apply the Python patch.

        patch -p1 < ${SLITHER_DIR}/patches/python-2.7.3.patch

3. Configure Python. At a minumum you'll need the following arguments:

        ./configure --prefix=[...] --disable-shared SO=.a DYNLOADFILE=dynload_redstorm.o

4. Copy `Modules/Setup.dist` to `Modules/Setup` and edit it, uncommenting
   all lines for modules which you would like to build into the base
   Python library.

5. Build and install Python.

        make
        make install


Using Slither
-------------

Prepare a slither configuration file. Each configuration file has the
following form:

    [DEFAULT]
    # Global options.

    # If True, byte-compile and include each `.py` in the generated
    # executable. Default is False.
    freeze = True

    [python]
    # Output executable name
    exe_name = mypython

    # If True, avoid as many filesystem calls as possible.
    # Equivalent to setting PYTHONHOME=/dev/null and calling Python
    # with the -S and -s flags. Default is False.
    # deep_freeze = True

    # List of standard library packages to exclude.
    excludes = test

    # compiler = cc
    # linker = cc
    # extra_link_args =

    # Python packages follow.

    # Sample format for each package:
    #
    #   [description]
    #   # Path, as would be added to `sys.path`
    #   path = .../path/to/.../lib/python2.7/site-packages
    #   # extra_link_args =
    #   # excludes =
    #
    # For example, to include NumPy and mpi4py:

    [numpy]
    path = .../numpy/1.7.1/lib/python2.7/site-packages
    extra_link_args = -L%(path)s/numpy/core/lib -lnpysort -lnpymath

    [mpi4py]
    path = .../mpi4py/1.3/lib/python2.7/site-packages


The custom Python executable can then be built using

    slither mypython.conf

It is important that the default Python executable is your custom
static installation. Otherwise Slither must be called as:

    .../bin/python .../bin/slither mypython.conf

The first line of the `slither` script may also be updated to point to
the static Python build.  (This will likely be done automatically
at a later date once a distutils `setup.py` script is provided for
slither).

See the `examples` directory for several example configuration
scripts.


Python Packages
===============

With few exceptions, Python packages should be built and installed as
usual.

    python setup.py build
    python setup.py install --prefix=...

Always use the `--prefix` option to allow additional granularity
when installing a package. For example, create a base python packages
directory and install into `.../package/version` as in
`python setup.py install --prefix=${PYTHON_PACKAGES}/numpy/1.7.1`.

Some Python packages require a certain amount of bootstrapping, in the
sense that you may first need to use slither to compile a version of
Python which contains Cython and NumPy before attempting to build that
new Python module.


Cython
------

Cython can be installed in a pure Python form. This makes the Cython
compilation stage marginally slower, but is much easier from a build
perspective.  To do this, add the `--no-cython-compile` flag to the
build and install stages.

NumPy
-----

NumPy 1.7.1 works out of the box, but a few patches may be required
for optimal performance.  In general you can build and install NumPy
1.7.1 as:

    python setup.py build
    python setup.py install --prefix=.../numpy/1.7.1
    cp build/temp.linux-x86_64-2.7/libnpysort.a .../numpy/1.7.1/lib/python2.7/site-packages/numpy/core/lib

If you plan on building any modules which use the `f2py` functionality
that NumPy provides it is important to first edit the
`numpy/f2py/src/fortranobject.h` file to redefine
`PY_ARRAY_UNIQUE_SYMBOL`.  See [NumPy #3014](https://github.com/numpy/numpy/pull/3014).

You may also want to apply the following patches:

* `patches/numpy-1.7.1-dotblas.patch`: Force the `_dotblas` module
  to be built. This lets NumPy use BLAS optimized matrix multiply
  operations. (Requires CBLAS).
* `patches/numpy-1.7.1-lapack.patch`: Use optimized `LAPACK`.

mpi4py
------

The `release-1.3` mercurial branch of `mpi4py` works without any
changes.  The `1.3` release will likely also work, but it has not
been tested.

h5py
----

The `2.1.2` release of `h5py` works without issue.

SciPy
-----

SciPy is difficult to get working. A regular install causes lots of
duplicate symbols, many of which can be safely ignored with an
appropriate linker flag.  (For example, `extra_link_args =
-Wl,-z,muldefs`).

Ignoring duplicate symbols may result in some bugs. Caveat emptor.
