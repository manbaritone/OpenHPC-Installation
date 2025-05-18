Amber is a suite of biomolecular simulation programs. It began in the late 1970's, and is maintained by an active development community.

The term "Amber" refers to two things. First, it is a set of molecular mechanical force fields for the simulation of biomolecules (these force fields are in the public domain, and are used in a variety of simulation programs). Second, it is a package of molecular simulation programs which includes source code and demos.

Amber is distributed in two parts: AmberTools24 and Amber24. You can use AmberTools24 without Amber24, but not vice versa.

Ref: https://ambermd.org/


### Pre-installation

OS System: CentOS Linux release 7.6.1810 with CMake 3.27.6, CUDA 11.8, GCC 7.3, and OpenMPI 4.1.6 operated by Lmod \
Computer System (Frontend node): 2 x NVIDIA GeForce GTX 1070 Ti 8GB with Intel(R) Core(TM) i7-7700K CPU @ 4.20GHz

Refs: https://ambermd.org/doc12/Amber24.pdf

#### Install Dependencies (For CentOS 7)
```
yum -y install tcsh make \
			   gcc gcc-gfortran gcc-c++ \
			   which flex bison patch bc \
			   libXt-devel libXext-devel \
			   perl perl-ExtUtils-MakeMaker util-linux wget \
			   bzip2 bzip2-devel zlib-devel tar
```
Other OS, please visit: https://ambermd.org/Installation.php

#### Extract AMBER
```
tar xvfj AmberTools24.tar.bz2
tar xvfj Amber24.tar.bz2 
```

#### Upgrade and update Amber
```
cd amber24_src
./update_amber --update
./update_amber --upgrade
```

### Compile Serial CPU

#### Compile and install
```
cd amber24_src
cd build
cmake .. -DCMAKE_INSTALL_PREFIX=/XXX/amber24 -DCOMPILER=GNU -DMPI=FALSE -DCUDA=FALSE -DINSTALL_TESTS=TRUE -DDOWNLOAD_MINICONDA=TRUE
make -j16 && make install
```

### Compile for Serial GPU

#### Compile and install
```
cd amber24_src
cd build
make clean
cmake .. -DCMAKE_INSTALL_PREFIX=/XXX/amber24 -DCOMPILER=GNU -DMPI=FALSE -DCUDA=TRUE -DCUDA_TOOLKIT_ROOT_DIR=/usr/local/cuda-11.8 -DINSTALL_TESTS=TRUE -DDOWNLOAD_MINICONDA=TRUE
make -j16 && make install
```

### Compile for Parallel CPU

#### Compile and install
```
cd amber24_src
cd build
make clean
cmake .. -DCMAKE_INSTALL_PREFIX=/XXX/amber24 -DCOMPILER=GNU -DCUDA=FALSE -DMPI=TRUE -DINSTALL_TESTS=TRUE -DDOWNLOAD_MINICONDA=TRUE
make -j16 && make install
```

### Compile for Parallel GPU

#### Compile and install
```
cd amber24_src
cd build
make clean
cmake .. -DCMAKE_INSTALL_PREFIX=/XXX/amber24 -DCOMPILER=GNU -DMPI=TRUE -DCUDA=TRUE -DCUDA_TOOLKIT_ROOT_DIR=/usr/local/cuda-11.8 -DINSTALL_TESTS=TRUE -DDOWNLOAD_MINICONDA=TRUE
make -j16 && make install
```
