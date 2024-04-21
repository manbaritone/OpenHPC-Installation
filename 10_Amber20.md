Amber is a suite of biomolecular simulation programs. It began in the late 1970's, and is maintained by an active development community.

The term "Amber" refers to two things. First, it is a set of molecular mechanical force fields for the simulation of biomolecules (these force fields are in the public domain, and are used in a variety of simulation programs). Second, it is a package of molecular simulation programs which includes source code and demos.

Amber is distributed in two parts: AmberTools20 and Amber20. You can use AmberTools20 without Amber20, but not vice versa.

Ref: https://ambermd.org/


### Pre-installation

System: CentOS Linux release 7.6.1810 with CMake 3.20.1, CUDA 11.1.1, GNU10 v10.3.0, OpenMPI4 v4.1.1 operated by Lmod

Refs: \
https://www.hull1.com/linux/2020/08/21/complie-amber20.html \
https://ambermd.org/doc12/Amber20.pdf

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
tar xvfj AmberTools20.tar.bz2
tar xvfj Amber20.tar.bz2
```

#### Create amber20 folder at destination path (e.g. /apps/amber20)
```
mkdir /apps/amber20
```

#### Upgrade and update Amber
```
cd amber20_src
./update_amber --upgrade
./update_amber --update
```

### Compile Serial CPU

#### Complie and install
```
cd amber20_src
cd build
cmake .. -DCMAKE_INSTALL_PREFIX=/apps/amber20 -DCOMPILER=GNU -DCUDA=FALSE -DINSTALL_TESTS=FALSE -DDOWNLOAD_MINICONDA=TRUE -DMINICONDA_USE_PY3=TRUE
make install -j8
```

#### Test
```
cd /apps/amber20
source amber.sh
make test.serial
```

### Compile for Serial GPU

Ref: https://ambermd.org/GPUHardware.php

Amber tries to support all CUDA SDK versions up to 11.x. In the past, they have recommended CUDA 9.1 or 9.2 for the best speed of the resulting executables, but this needs to be revisited. Here are the minimum requirements for different tiers of hardware:
* Ampere (SM_80) based cards require CUDA 11.0 or later (RTX-3060, RTX-3070, RTX-3080, RTX-3090, RTX-A6000, A100).
* Turing (SM_75) based cards require CUDA 9.2 or later (RTX-2080Ti, RTX-2080, RTX-2070, Quadro RTX6000/5000/4000).
* Volta (SM_70) based cards require CUDA 9.0 or later (Titan-V, V100, Quadro GV100).
* Pascal (SM_61) based cards require CUDA 8.0 or later. (Titan-XP [aka Pascal Titan-X], GTX-1080TI / 1080 / 1070 / 1060, Quadro P6000 / P5000, P4 / P40).

* GTX-1080 cards require NVIDIA Driver version >= 367.27 for reliable numerical results.
* GTX-Titan and GTX-780 cards require NVIDIA Driver version >= 319.60 for correct numerical results.
* GTX-780Ti cards require a modified Bios from Exxact Corp to give correct numerical results.
* GTX-Titan-Black Edition cards require NVIDIA Driver version >= 337.09 or 331.79 or later for correct numerical results.

#### Complie and install
```
cd amber20_src
cd build
make clean
cmake .. -DCMAKE_INSTALL_PREFIX=/apps/amber20 -DCOMPILER=GNU -DCUDA=TRUE -DCUDA_TOOLKIT_ROOT_DIR=/usr/local/cuda-9.2 -DINSTALL_TESTS=FALSE -DDOWNLOAD_MINICONDA=TRUE -DMINICONDA_USE_PY3=TRUE
make install -j8
```

#### Test
```
cd /apps/amber20
source amber.sh
make test.cuda
```

### Compile for Parallel CPU

#### Complie and install
```
cd amber20_src
cd build
make clean
cmake .. -DCMAKE_INSTALL_PREFIX=/apps/amber20 -DCOMPILER=GNU -DCUDA=FALSE -DMPI=TRUE -DINSTALL_TESTS=FALSE -DDOWNLOAD_MINICONDA=TRUE -DMINICONDA_USE_PY3=TRUE
make install -j8
```

#### Test
```
cd /apps/amber20
export DO_PARALLEL='mpirun -np 2'
test -f amber.sh  && source amber.sh
export DO_PARALLEL='mpirun -np 2'
make test.parallel
```

### Compile for Parallel GPU

Ref: https://ambermd.org/GPUHardware.php

Amber tries to support all CUDA SDK versions up to 11.x. In the past, they have recommended CUDA 9.1 or 9.2 for the best speed of the resulting executables, but this needs to be revisited. Here are the minimum requirements for different tiers of hardware:
* Ampere (SM_80) based cards require CUDA 11.0 or later (RTX-3060, RTX-3070, RTX-3080, RTX-3090, RTX-A6000, A100).
* Turing (SM_75) based cards require CUDA 9.2 or later (RTX-2080Ti, RTX-2080, RTX-2070, Quadro RTX6000/5000/4000).
* Volta (SM_70) based cards require CUDA 9.0 or later (Titan-V, V100, Quadro GV100).
* Pascal (SM_61) based cards require CUDA 8.0 or later. (Titan-XP [aka Pascal Titan-X], GTX-1080TI / 1080 / 1070 / 1060, Quadro P6000 / P5000, P4 / P40).

* GTX-1080 cards require NVIDIA Driver version >= 367.27 for reliable numerical results.
* GTX-Titan and GTX-780 cards require NVIDIA Driver version >= 319.60 for correct numerical results.
* GTX-780Ti cards require a modified Bios from Exxact Corp to give correct numerical results.
* GTX-Titan-Black Edition cards require NVIDIA Driver version >= 337.09 or 331.79 or later for correct numerical results.

#### Complie and install
```
cd amber20_src
cd build
make clean
cmake .. -DCMAKE_INSTALL_PREFIX=/apps/amber20 -DCOMPILER=GNU -DMPI=TRUE -DCUDA=TRUE -DCUDA_TOOLKIT_ROOT_DIR=/usr/local/cuda-9.2 -DINSTALL_TESTS=FALSE -DDOWNLOAD_MINICONDA=TRUE -DMINICONDA_USE_PY3=TRUE
make install -j8
```

#### Test
```
cd /apps/amber20
export DO_PARALLEL='mpirun -np 2'
test -f amber.sh  && source amber.sh
export DO_PARALLEL='mpirun -np 2'
make test.cuda.parallel
```

### Compile for Parallel GPU with NCCL

Ref: https://ambermd.org/GPUHardware.php

Amber tries to support all CUDA SDK versions up to 11.x. In the past, they have recommended CUDA 9.1 or 9.2 for the best speed of the resulting executables, but this needs to be revisited. Here are the minimum requirements for different tiers of hardware:
* Ampere (SM_80) based cards require CUDA 11.0 or later (RTX-3060, RTX-3070, RTX-3080, RTX-3090, RTX-A6000, A100).
* Turing (SM_75) based cards require CUDA 9.2 or later (RTX-2080Ti, RTX-2080, RTX-2070, Quadro RTX6000/5000/4000).
* Volta (SM_70) based cards require CUDA 9.0 or later (Titan-V, V100, Quadro GV100).
* Pascal (SM_61) based cards require CUDA 8.0 or later. (Titan-XP [aka Pascal Titan-X], GTX-1080TI / 1080 / 1070 / 1060, Quadro P6000 / P5000, P4 / P40).

* GTX-1080 cards require NVIDIA Driver version >= 367.27 for reliable numerical results.
* GTX-Titan and GTX-780 cards require NVIDIA Driver version >= 319.60 for correct numerical results.
* GTX-780Ti cards require a modified Bios from Exxact Corp to give correct numerical results.
* GTX-Titan-Black Edition cards require NVIDIA Driver version >= 337.09 or 331.79 or later for correct numerical results.

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
cd amber20_src
export NCCL_HOME="~/apps/lib/nccl_2.6.4-1+cuda10.0_x86_64"
cd build
make clean
cmake .. -DCMAKE_INSTALL_PREFIX=/apps/amber20 -DCOMPILER=GNU -DMPI=TRUE -DCUDA=TRUE -DCUDA_TOOLKIT_ROOT_DIR=/usr/local/cuda-9.2 -DNCCL=TRUE -DPnetCDF_C_LIBRARY=~/apps/pnetcdf/lib/ -DPnetCDF_C_INCLUDE_DIR=~/apps/pnetcdf/include/ -DINSTALL_TESTS=FALSE -DDOWNLOAD_MINICONDA=TRUE -DMINICONDA_USE_PY3=TRUE
make install -j8
```

#### Test
```
cd /apps/amber20
export DO_PARALLEL='mpirun -np 2'
test -f amber.sh  && source amber.sh
export DO_PARALLEL='mpirun -np 2'
make test.cuda.parallel
```
