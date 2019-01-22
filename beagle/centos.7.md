# Run MrBayes+BEAGLE3 inside CentOS 7 Docker container

- Last modified: Tue Jan 22, 2019  01:18PM
- Sign: JN
- Status: CUDA:NA, OpenCL:FAIL, OpenMPI:OK, BEAGLE3:OK, MB-MPI:OK, MB:OK
- Comment: Can't get beagle to work from inside mb unless beagle is configured using `-rpath`, or, `LD_LIBRARY_PATH` is set before running mb. And, can't get beagle to run if OpenCL is installed.

# Install and run

    $ docker run --dns 172.16.0.9 -it debian:sid /bin/bash

    # Base system
    mywd=$(pwd)

    # Base system
    yum -y update
    yum -y install \
        libtool \
        autoconf \
        make \
        gcc-c++ \
        git \
        java-1.8.0-openjdk-devel \
        texlive

    # Note: Currently (18 Jan 2019), Beagle seems not to work if OpenCL is
    # installed. Not even when using `./configure --without-opencl`.
    # Beagle-error message when running `mb`:
    # "Unknown error from file <GPUInterfaceOpenCL.cpp>, line 115."
    #
    # OpenCL. Apparently, need to install "EPEL" sources to get OpenCL
    #yum -y install https://dl.fedoraproject.org/pub/epel/epel-release-latest-7.noarch.rpm
    #yum -y install \
    #    opencl-headers \
    #    ocl-icd \
    #    ocl-icd-devel
    #ln -s /usr/lib64/libOpenCL.so.1 /usr/lib/libOpenCL.so # Hack https://unix.stackexchange.com/questions/292630/how-to-install-opencl-in-centos-7-using-yum

    # OpenMPI. Apparently, mpirun need to be handled by the `module` system.
    yum -y install \
        openmpi \
        openmpi-devel \
        environment-modules
    source /etc/profile
    module load mpi/openmpi-x86_64

    # BEAGLE
    cd "$mywd"
    git clone --depth=1 https://github.com/beagle-dev/beagle-lib.git
    cd beagle-lib
    ./autogen.sh
    ./configure LDFLAGS=-Wl,-rpath,/usr/local/lib
    make
    make install

    # MrBayes
    cd "$mywd"
    git clone https://github.com/NBISweden/MrBayes.git
    cd MrBayes
    git checkout feature/beagle3
    cd "$mywd"
    git clone https://github.com/NBISweden/MrBayes.git
    cd MrBayes
    git checkout feature/beagle3
    ./configure --with-mpi
    make
    mpirun --allow-run-as-root -np 1 src/mb <<MBCMD
    showb
    quit
    MBCMD
    make clean
    ./configure
    make
    src/mb <<MBCMD
    showb
    quit
    MBCMD
