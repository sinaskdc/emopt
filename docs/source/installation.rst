.. _installation_instructions:

#########################
Installation Instructions
#########################

These instructions will help you get EMopt up and running on your system.
Currently the installation requires some manual labor.

======================
Software Prerequisites
======================

EMopt depends on a variety of software packages/libraries. Before using EMopt,
these packages must be installed:

-------------------
C/C++ Requirements:
-------------------
* `Eigen <http://eigen.tuxfamily.org/>`_: A library for handling arrays and
  matrices
* `boost.geometry <http://www.boost.org/doc/libs/develop/libs/geometry/doc/html/index.html)>`_:
  For performing computational geometry operations.
* `PETSc <https://www.mcs.anl.gov/petsc/>`_: For solving large systems of the form Ax=b.
* `SLEPc <http://slepc.upv.es/>`_: For solving large eigenvalue problems of the form Ax=nBx.

--------------------
Python Requirements:
--------------------
* `Python 2.7+ <https://www.python.org/>`_
* `numpy <http://www.numpy.org/>`_
* `scipy <https://www.scipy.org/>`_
* `mpi4py <http://mpi4py.scipy.org/docs/>`_: For handling paralellism
* `petsc4py <https://pypi.python.org/pypi/petsc4py>`_: Python interface for PETSc for solving large linear systems of
  equations
* `slepc4py <https://pypi.python.org/pypi/slepc4py>`_: Python interface for SLEPc for solving large eigenvalue
  problems.

------------------------
Optional Python Packages
------------------------

These packages are not required for the core functionality of EMopt but
provided useful functionality which is leveraged within some of EMopt's modules
(e.g. plotting, defining complex geometries, etc).

* `h5py <http://www.h5py.org/>`_: For saving and loading simulation data.
* `matplotlib <https://matplotlib.org/>`_: For plotting optimization results
* `shapely <https://github.com/Toblerity/Shapely>`_: For handling more complicated geometric operations

In addition to these specific packages, your system must be 
equipped with the ability to compile c++ source using a makefile.

In the following sections, we will work through installing these prerequisite
packages on a common Linux system (Cent OS 7 in this case).

==============================
Installing C/C++ Prerequisites
==============================

Depending on the system you are installing EMopt on, you may wish to install
it such that everyone on the system can use it or only a single user can use
it.  In this guide, we will assume that EMopt will be installed for a local
user. This process is easily modified to work for system-wide installation.

All prerequiste libraries will be installed to a folder ``$HOME/local`` where
``$HOME`` is the path to your (the user's) home directory. Inside of this global
installation directory, we will also need to create a subdirectory where the
required header-only libraries will be stored.

------------------------
Installing Eigen Headers
------------------------

Eigen is a header-only library and does not need to be compiled. Simply
download the most recent stable release from
`here <http://eigen.tuxfamily.org/index.php?title=Main_Page>`_. Next, unpack the
tarball and copy the folder containing the header files to the desired directory
in ``$HOME/local``, e.g.
::
    $ curl -L -O http://bitbucket.org/eigen/eigen/get/3.3.4.tar.gz
    $ tar xvzf 3.3.4.tar.gz
    $ cp -r eigen-eigen-5a0156e40feb/Eigen $HOME/local/include/Eigen/

------------------------
Installing Boost Headers
------------------------

Boost.Geometry is also header-only and does not need to be compiled. Simply
download the most recent stable release of the boost libraries from 
`here <http://www.boost.org/users/download/>`_ and copy the boost folder from
the unpacked contents to the desired folder in $HOME/local, e.g.

::

    $ curl -L -O https://dl.bintray.com/boostorg/release/1.65.1/source/boost_1_65_1.tar.gz
    $ tar xvzf boost_1_65_1.tar.gz
    $ cp -r boost_1_65_1/boost/ $HOME/local/include/boost/

---------------
Compiling PETSc
---------------

PETSc is a powerful library for solving large sparse systems in a distributed
manner. Before using EMopt, PETSc must be compiled and installed on your
system.  In order for PETSc to work with EMopt, we need to be very careful
to compile PETSc with support for complex scalars and include support for
scalapack and MUMPS. Luckily, the PETSc compilation scripts make this relatively 
easy for us. 

Just a word of warning: getting PETSc compiled with exactly the desired
configuration can be a bit tricky at times and may require some
experimentation.

Before compiling PETSc, we first need to install an MPI implementation.
Furthermore, you may wish to install additional PETSc dependencies (BLAS,
lapack, scalapack, and MUMPS) through your package manager; for simplicity, we
just let PETSc's configure script take care of these remaining dependencies.

In this case, we will install openmpi, however mpich should work equally well. 
In most Linux distributions, openmpi can be installed using the package manager. 
On yum-base systems we run::

    $ sudo yum install openmpi openmpi-devel

Depending on your system, you may also need to load the openmpi module::

    $ sudo module load openmpi-x86_64

With yum installed, we are ready to install PETSc. First, download the current
stable version of PETSc, e.g.
::

    $ curl -L -O http://ftp.mcs.anl.gov/pub/petsc/release-snapshots/petsc-3.8.0.tar.gz

Next, unzip the contents of the tarball and move into the newly created
directory::

    $ tar xvzf petsc-3.8.0.tar.gz
    $ cd petsc-3.8.0/

Before moving ahead with the compilation, we first need to create a directory
where the compiled library will be installed. A good choice is in the
$HOME/local directory that we created at the beginning::

    $ mkdir $HOME/local/petsc/
    $ mkdir $HOME/local/petsc/petsc-3.8.0/

Having created an installation directory, we can move ahead with the
compilation of PETSc. Configure the project for compilation by running::

    $ ./configure --with-scalar-type=complex --with-mpi=1 --COPTFLAGS='-O3' \
    $ --FOPTFLAGS='-O3' --CXXOPTFLAGS='-O3' --with-debugging=0 \
    $ --prefix=$HOME/local/petsc/petsc-3.8.0 --download-scalapack --download-mumps \
    $ --download-openblas

There a number of important things to note here. First, depending on your
system and whether you chose openmpi or mpich, the path following
``---with-mpi-dir`` may need to be modified. Next, depending on the version of
petsc that you are compiling the ``--prefix=`` path may need modification.
Furthermore, we have chosen to allow PETSc to handle the compilation of a
number of important dependencies. If you wish to use packages installed by your
package manager, these options will need to be modified. 
`Consult the PETSc installation manual for details. <https://www.mcs.anl.gov/petsc/documentation/installation.html>`_

After this step has completed (which may take a few minutes), the script should
tell you the command to run to compile PETSc. It should look something like::

    $ make PETSC_DIR=$HOME/Downloads/petsc-3.8.0 PETSC_ARCH=arch-linux2-c-opt all

Run this command and verify that it completes successfully. The output should
tell you the command needed to complete the installation of PETSc. In my case::

    $ make PETSC_DIR=$HOME/Downloads/petsc-3.8.0 PETSC_ARCH=arch-linux2-c-opt install

At this point, PETSc should be installed. The installation script will present
you with additional commands to run to check that the library has been compiled
and installed succcessfully. This is not a bad idea.

Note: compiling PETSc with ``--with-clanguage=cxx`` will likely prevent the installation of slepc4py from working.

---------------
Compiling SLEPc
---------------

SLEPc is a library for solving large sparse eigenvalue problems. Because it is
built on top of PETSc, there are no additional dependencies that are needed.

To begin, download the most recent stable release from `here <http://slepc.upv.es/download/>`_
and unpack the contents::

    $ curl -L -O http://slepc.upv.es/download/distrib/slepc-3.8.1.tar.gz
    $ tar xvzf slepc-3.8.1.tar.gz
    $ cd slepc-3.8.1/

Before we can build SLEPc, we need to tell it where to find PETSc. We do this by
defining the appropriate environment variable::

    $ export PETSC_DIR=$HOME/local/petsc/petsc-3.8.0/

Furthermore, we need to create the appropriate directory where SLEPc will be
installed::

    $ mkdir $HOME/local/slepc
    $ mkdir $HOME/local/slepc/slepc-3.8.1/

Finally, we can go ahead and configure, make, and test the SLEPc installation::

    $ ./configure --prefix=$HOME/local/slepc/slepc-3.8.1/

As with PETSc, SLEPc's make scripts will tell you the next steps. Execute the make
commands that it tells you. For example, the commands should look like::

    $ make SLEPC_DIR=$PWD PETSC_DIR=$HOME/local/petsc/petsc-3.8.0
    $ make SLEPC_DIR=$HOME/Downloads/slepc-3.8.1 PETSC_DIR=$HOME/local/petsc/petsc-3.8.0 install
    $ make SLEPC_DIR=$HOME/local/slepc/slepc-3.8.1 PETSC_DIR=$HOME/local/petsc/petsc-3.8.0 PETSC_ARCH="" test

===============================
Installing Python Prerequisites
===============================

.. note::
    this assumes that you already have python 2.7+, pip, and the python
    development libraries (e.g. python-devel) installed on your system.

Before using EMopt, we need to install numpy, scipy, mpi4py, petsc4py, and
slepc4py. Additionally, it is strongly recommended that you install h5py and
matplotlib. 

Numpy, scipy, and mpi4py can be installed in a variety of ways. Here, we use 
`pip <https://packaging.python.org/tutorials/installing-packages/>`_::

    $ pip install --user numpy
    $ pip install --user scipy
    $ pip install --user mpi4py

To install petsc4py, we need to ensure that the environment variable ``PETSC_DIR`` is
to our PETSc installation directory::

    $ export PETSC_DIR=$HOME/local/petsc/petsc-3.8.0

Next, install petsc4py using pip::
    
    $ pip install --user petsc4py

slepc4py is installed in a similar manner. Once again, we must be sure to set an
environment variable ``SLEPC_DIR`` such that it points to our SLEPc installation
directory::

    $ export SLEPC_DIR=$HOME/local/slepc/slepc-3.8.1/
    $ pip install --user slepc4py

Finally, if desired, install matplotlib and h5py::

    $ pip install --user h5py
    $ pip install --user matplotlib

================
Installing EMopt
================

Once the repository has been cloned (or downloaded), change into the emopt
directory, and make build EMopt using make. Before running make, we need to
tell EMopt where to find the include files for eigen and boost. We do this by
setting the appropriate environment variables::

    $ export EIGEN_DIR=$HOME/local/include/
    $ export BOOST_DIR=$HOME/local/include/

In our case, both libraries are installed in the same directory, so these two
environment variables have the same path. This may not be the case on your
system, however.

Next, run the setup.py script to build and install EMopt::

    $ python setup.py install --user

Assuming this completes without error, you should be all set and ready to go!

To learn how to use EMopt, head over to the :ref:`tutorials
section<tutorials_main>` section.

======================
A Note on MPI + OpenMP
======================

By default, emopt (and its dependencies) will use OpenMP to further parallelize some
tasks. Unfortunately, on many systems the number of threads used for OpenMP will
default to the number of cores available. This is problematic when using more than
one process for MPI as emopt will try to use more threads than cores in the machine,
leading to slow performance. 

In order to avoid this, when running emopt on a single machine, it is advisable to
set the number of OpenMP threads to 1 using::

    $ export OMP_NUM_THREADS=1
    $ mpirun -n 12 python code_to_run.py

or::

    $ OMP_NUM_THREADS=1 mpirun -n 12 python code_to_run.py

If running on a network/cluster, increasing the number of threads used by OpenMP
should be fine.
