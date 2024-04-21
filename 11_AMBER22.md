Amber is a suite of biomolecular simulation programs. It began in the late 1970's, and is maintained by an active development community.

The term "Amber" refers to two things. First, it is a set of molecular mechanical force fields for the simulation of biomolecules (these force fields are in the public domain, and are used in a variety of simulation programs). Second, it is a package of molecular simulation programs which includes source code and demos.

Amber is distributed in two parts: AmberTools22 and Amber22. You can use AmberTools22 without Amber22, but not vice versa.

Ref: https://ambermd.org/


### Pre-installation

System: CentOS Linux release 7.6.1810 with CMake 3.27.6, CUDA 11.8, GNU11 v11.2.1, and OpenMPI4 v4.1.6 operated by Lmod

Refs: \
https://ambermd.org/doc12/Amber22.pdf

#### Install Dependencies
```
yum -y install tcsh make \
			   gcc gcc-gfortran gcc-c++ \
			   which flex bison patch bc \
			   libXt-devel libXext-devel \
			   perl perl-ExtUtils-MakeMaker util-linux wget \
			   bzip2 bzip2-devel zlib-devel tar
```

#### Extract amber
```
tar xvfj AmberTools23.tar.bz2
tar xvfj Amber22.tar.bz2
```

#### Upgrade and update Amber
```
cd amber22_src
./update_amber --update
```
if have any error due to "https", please modify the code in 4 py files (downloader.py, main.py, patch.py, and test_updateutils.py) in amber22_src/updateutils folder from "https" to "http". You can easily modify by using the command in VIM with ":%s/https/http".


### Compile Serial CPU

#### Complie and install
```
cd amber22_src
cd build
cmake .. -DCMAKE_INSTALL_PREFIX=/apps/amber22 -DCOMPILER=GNU -DCUDA=FALSE -DMPI=FALSE -DINSTALL_TESTS=FALSE -DDOWNLOAD_MINICONDA=TRUE -DMINICONDA_USE_PY3=TRUE -DMINICONDA_VERSION=py38_4.12.0
make install -j8
```

#### Test
```
cd /apps/amber22
source amber.sh
make test.serial
```

### Compile for Serial GPU

Ref: https://ambermd.org/GPUHardware.php

#### Complie and install
```
cd amber22_src
cd build
make clean
cmake .. -DCMAKE_INSTALL_PREFIX=/apps/amber22 -DCOMPILER=GNU -DMPI=FALSE -DCUDA=TRUE -DCUDA_TOOLKIT_ROOT_DIR=/usr/local/cuda-11.8 -DINSTALL_TESTS=FALSE -DDOWNLOAD_MINICONDA=TRUE -DMINICONDA_USE_PY3=TRUE -DMINICONDA_VERSION=py38_4.12.0
make install -j8
```

#### Test
```
cd /apps/amber22
source amber.sh
make test.cuda.serial
```

### Compile for Parallel CPU

#### Complie and install
```
cd amber22_src
cd build
make clean
cmake .. -DCMAKE_INSTALL_PREFIX=/apps/amber22 -DCOMPILER=GNU -DCUDA=FALSE -DMPI=TRUE -DINSTALL_TESTS=FALSE -DDOWNLOAD_MINICONDA=TRUE -DMINICONDA_USE_PY3=TRUE -DMINICONDA_VERSION=py38_4.12.0
make install -j8
```

#### Test
```
cd /apps/amber22
export DO_PARALLEL='mpirun -np 2'
test -f amber.sh  && source amber.sh
export DO_PARALLEL='mpirun -np 2'
make test.parallel
```

### Compile for Parallel GPU

Ref: https://ambermd.org/GPUHardware.php

#### Complie and install
```
cd amber22_src
cd build
make clean
cmake .. -DCMAKE_INSTALL_PREFIX=/apps/amber22 -DCOMPILER=GNU -DMPI=TRUE -DCUDA=TRUE -DCUDA_TOOLKIT_ROOT_DIR=/usr/local/cuda-11.8 -DINSTALL_TESTS=FALSE -DDOWNLOAD_MINICONDA=TRUE -DMINICONDA_USE_PY3=TRUE -DMINICONDA_VERSION=py38_4.12.0
make install -j8
```

#### Test
```
cd /apps/amber22
export DO_PARALLEL='mpirun -np 2'
test -f amber.sh  && source amber.sh
export DO_PARALLEL='mpirun -np 2'
make test.cuda.parallel
```

### Compile for Parallel GPU with NCCL

Ref: https://ambermd.org/GPUHardware.php

#### Download NVIDIA NCCL from https://developer.nvidia.com/nccl

#### Install PnetCDF

* Download PnetCDF from https://parallel-netcdf.github.io/wiki/Download.html
```
autoreconf -i
./configure --prefix=/apps/PnetCDF
make -j8
make install
```

#### Complie and install
```
cd amber22_src
export NCCL_HOME="~/apps/lib/nccl_2.6.4-1+cuda10.0_x86_64"
cd build
make clean
cmake .. -DCMAKE_INSTALL_PREFIX=/apps/amber22 -DCOMPILER=GNU -DMPI=TRUE -DCUDA=TRUE -DCUDA_TOOLKIT_ROOT_DIR=/usr/local/cuda-11.8 -DNCCL=TRUE -DPnetCDF_C_LIBRARY=~/apps/pnetcdf/lib/ -DPnetCDF_C_INCLUDE_DIR=~/apps/pnetcdf/include/ -DINSTALL_TESTS=FALSE -DDOWNLOAD_MINICONDA=TRUE -DMINICONDA_USE_PY3=TRUE -DMINICONDA_VERSION=py38_4.12.0
make install -j8
```

#### Test
```
cd /apps/amber22
export DO_PARALLEL='mpirun -np 2'
test -f amber.sh  && source amber.sh
export DO_PARALLEL='mpirun -np 2'
make test.cuda.parallel
```
