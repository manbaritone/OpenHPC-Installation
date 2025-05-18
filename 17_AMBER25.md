Amber is a suite of biomolecular simulation programs. It began in the late 1970's, and is maintained by an active development community.

The term "Amber" refers to two things. First, it is a set of molecular mechanical force fields for the simulation of biomolecules (these force fields are in the public domain, and are used in a variety of simulation programs). Second, it is a package of molecular simulation programs which includes source code and demos.

Starting in 2025, Amber and AmberTools are separate releases, residing in different folders. Each can be compiled on its own, without the other. One improvement is that compilation of just the pmemd program (say on a high-performance computer system) is now much simpler in the past. A second advantage for many users will be this: AmberTools is available as a pre-compiled conda package (see ambermd.org/GetAmber.php) which can be installed via “conda install”. Users who choose this option only need to download and compile pmemd24.tar.bz2.

**Ref:** https://ambermd.org/ and https://ambermd.org/doc12/Amber25.pdf \
**Download AmberTools25 and Amber24:** https://ambermd.org/GetAmber.php

### Pre-installation

OS System: CentOS Linux release 7.6.1810 with CMake 3.20.1, CUDA 11.1.1, GCC 10.3.0, and OpenMPI 4.1.1 operated by Lmod Computer \
System: 4 x Tesla V100 SXM2 32GB with Intel(R) Xeon(R) Gold 6248 CPU @ 2.50GHz

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

#### Load modules
```
ml CMake/3.20.1-GCCcore-10.3.0
ml OpenMPI/4.1.1-GCC-10.3.0
ml CUDAcore/11.1.1
```

#### Extract AMBER
```
tar xvfj ambertools25.tar.bz2
tar xvfj pmemd24.tar.bz2 
```

### AmberTools25 Installation
**Note: please define destination folder for -DCMAKE_INSTALL_PREFIX flag**

#### Upgrade and update Amber
```
cd ambertools25_src
./update_amber --update
./update_amber --upgrade
```

#### Compile and install with Serial CPU
```
cd ambertools25_src
cd build
cmake .. -DCMAKE_INSTALL_PREFIX=/XXX/amber24_abt25 -DCOMPILER=GNU -DMPI=FALSE -DCUDA=FALSE -DINSTALL_TESTS=TRUE -DDOWNLOAD_MINICONDA=TRUE
make -j32 && make install
```

#### Compile and install with Serial GPU
```
cd ambertools25_src
cd build
make clean
cmake .. -DCMAKE_INSTALL_PREFIX=/XXX/amber24_abt25 -DCOMPILER=GNU -DMPI=FALSE -DCUDA=TRUE -DINSTALL_TESTS=TRUE -DDOWNLOAD_MINICONDA=TRUE
make -j32 && make install
```

#### Compile and install with Parallel CPU
```
cd ambertools25_src
cd build
make clean
cmake .. -DCMAKE_INSTALL_PREFIX=/XXX/amber24_abt25 -DCOMPILER=GNU -DCUDA=FALSE -DMPI=TRUE -DINSTALL_TESTS=TRUE -DDOWNLOAD_MINICONDA=TRUE
make -j32 && make install
```

#### Compile and install with Parallel GPU
```
cd ambertools25_src
cd build
make clean
cmake .. -DCMAKE_INSTALL_PREFIX=/ist/users/bunditb/bunditb_bak2/apps/amber24_abt25 -DCOMPILER=GNU -DMPI=FALSE -DCUDA=FALSE -DINSTALL_TESTS=TRUE -DDOWNLOAD_MINICONDA=FALSE -DBUILD_PYTHON=FALSE -DBUILD_PERL=FALSE -DBUILD_GUI=FALSE -DPMEMD_ONLY=TRUE -DCHECK_UPDATES=FALSE
make -j32 && make install
```

### Amber24 (pmemd24) Installation

#### Upgrade and update Amber
```
cd pmemd24_src
./update_amber --update
./update_amber --upgrade
```

#### Compile and install with Serial CPU
```
cd pmemd24_src
cd build
cmake .. -DCMAKE_INSTALL_PREFIX=/XXX/amber24_abt25 -DCOMPILER=GNU -DMPI=FALSE -DCUDA=FALSE -DINSTALL_TESTS=TRUE -DDOWNLOAD_MINICONDA=FALSE -DBUILD_PYTHON=FALSE -DBUILD_PERL=FALSE -DBUILD_GUI=FALSE -DPMEMD_ONLY=TRUE -DCHECK_UPDATES=FALSE
make -j32 && make install
```

#### Compile and install with Serial GPU
```
cd pmemd24_src
cd build
make clean
cmake .. -DCMAKE_INSTALL_PREFIX=/XXX/amber24_abt25 -DCOMPILER=GNU -DMPI=FALSE -DCUDA=TRUE -DINSTALL_TESTS=TRUE -DDOWNLOAD_MINICONDA=FALSE -DBUILD_PYTHON=FALSE -DBUILD_PERL=FALSE -DBUILD_GUI=FALSE -DPMEMD_ONLY=TRUE -DCHECK_UPDATES=FALSE
make -j32 && make install
```

#### Compile and install with Parallel CPU
```
cd pmemd24_src
cd build
make clean
cmake .. -DCMAKE_INSTALL_PREFIX=/XXX/amber24_abt25 -DCOMPILER=GNU -DCOMPILER=GNU -DMPI=TRUE -DCUDA=FALSE -DINSTALL_TESTS=TRUE -DDOWNLOAD_MINICONDA=FALSE -DBUILD_PYTHON=FALSE -DBUILD_PERL=FALSE -DBUILD_GUI=FALSE -DPMEMD_ONLY=TRUE -DCHECK_UPDATES=FALSE
make -j32 && make install
```

#### Compile and install with Parallel GPU
```
cd pmemd24_src
cd build
make clean
cmake .. -DCMAKE_INSTALL_PREFIX=/XXX/amber24_abt25 -DCOMPILER=GNU -DMPI=TRUE -DCUDA=TRUE -DINSTALL_TESTS=TRUE -DDOWNLOAD_MINICONDA=FALSE -DBUILD_PYTHON=FALSE -DBUILD_PERL=FALSE -DBUILD_GUI=FALSE -DPMEMD_ONLY=TRUE -DCHECK_UPDATES=FALSE
make -j32 && make install
```
