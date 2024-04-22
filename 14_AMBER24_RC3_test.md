## AMBER24_RC3_test

OS System: CentOS Linux release 7.6.1810 with CMake 3.20.1, CUDA 11.1.1, GCC 10.3.0, and OpenMPI 4.1.1 operated by Lmod
Computer System: 4 x Tesla V100 SXM2 32GB with Intel(R) Xeon(R) Gold 6248 CPU @ 2.50GHz

### Load modules
```
ml CMake/3.20.1-GCCcore-10.3.0
ml OpenMPI/4.1.1-GCC-10.3.0
ml CUDAcore/11.1.1
```

#### Extract AMBER
```
tar xvfj AmberTools24_rc3.tar.bz2
tar xvfj Amber24_rc3.tar.bz2 
```

### Compile Serial CPU

#### Complie and install
```
cd amber24_src
cd build
cmake .. -DCMAKE_INSTALL_PREFIX=/ist/users/bunditb/bunditb_bak2/apps/amber24 -DCOMPILER=GNU -DMPI=FALSE -DCUDA=FALSE -DINSTALL_TESTS=TRUE -DDOWNLOAD_MINICONDA=TRUE -DMINICONDA_USE_PY3=TRUE
make install -j16
```

#### Test
```
sinteractive -p gpu-cluster --gpus 1 --mem=32GB --cpus-per-gpu=1 --time=1-0:0:0
cd /ist/users/bunditb/bunditb_bak2/apps/amber24
source amber.sh
make test.serial
```

### Compile for Serial GPU

#### Complie and install
```
cd amber24_src
cd build
make clean
cmake .. -DCMAKE_INSTALL_PREFIX=/ist/users/bunditb/bunditb_bak2/apps/amber24 -DCOMPILER=GNU -DMPI=FALSE -DCUDA=TRUE -DCUDA_TOOLKIT_ROOT_DIR=/ist/apps/modules/software/CUDAcore/11.1.1 -DINSTALL_TESTS=TRUE -DDOWNLOAD_MINICONDA=TRUE -DMINICONDA_USE_PY3=TRUE
make install -j16
```

#### Test
```
sinteractive -p gpu-cluster --gpus 1 --mem=32GB --cpus-per-gpu=1 --time=1-0:0:0
cd /ist/users/bunditb/bunditb_bak2/apps/amber24
source amber.sh
make test.cuda.serial
```

### Compile for Parallel CPU

#### Complie and install
```
cd amber24_src
cd build
make clean
cmake .. -DCMAKE_INSTALL_PREFIX=/ist/users/bunditb/bunditb_bak2/apps/amber24 -DCOMPILER=GNU -DCUDA=FALSE -DMPI=TRUE -DINSTALL_TESTS=TRUE -DDOWNLOAD_MINICONDA=TRUE -DMINICONDA_USE_PY3=TRUE
make install -j16
```

#### Test for 2 MPI threads
```
sinteractive -p gpu-cluster --gpus 1 --mem=32GB --ntasks=2 --time=1-0:0:0
cd /ist/users/bunditb/bunditb_bak2/apps/amber24
source amber.sh
export DO_PARALLEL='mpirun -np 2'
make test.parallel
```

#### Test for 4 MPI threads
```
sinteractive -p gpu-cluster --gpus 1 --mem=32GB --ntasks=4 --time=1-0:0:0
cd /ist/users/bunditb/bunditb_bak2/apps/amber24
source amber.sh
export DO_PARALLEL='mpirun -np 4'
make test.parallel
```

### Compile for Parallel GPU

#### Complie and install
```
cd amber24_src
cd build
make clean
cmake .. -DCMAKE_INSTALL_PREFIX=/ist/users/bunditb/bunditb_bak2/apps/amber24 -DCOMPILER=GNU -DMPI=TRUE -DCUDA=TRUE -DCUDA_TOOLKIT_ROOT_DIR=/usr/local/cuda-11.8 -DINSTALL_TESTS=TRUE -DDOWNLOAD_MINICONDA=TRUE -DMINICONDA_USE_PY3=TRUE
make install -j16
```

#### Test for 2 MPI threads
```
sinteractive -p gpu-cluster --nodes=1 --gres=gpu:1 --mem=32GB --ntasks=2 --time=1-0:0:0
cd /ist/users/bunditb/bunditb_bak2/apps/amber24
source amber.sh
export DO_PARALLEL='mpirun -np 2'
make test.cuda.parallel
```

#### Test for 4 MPI threads
```
sinteractive -p gpu-cluster --nodes=1 --gres=gpu:1 --mem=32GB --ntasks=4 --time=1-0:0:0
cd /ist/users/bunditb/bunditb_bak2/apps/amber24
source amber.sh
export DO_PARALLEL='mpirun -np 4'
make test.cuda.parallel
```
