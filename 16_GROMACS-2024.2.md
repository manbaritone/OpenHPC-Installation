GROMACS (GROningen MAchine for Chemical Simulations) is a high-performance molecular dynamics simulation software. It is primarily designed for simulating the interactions between molecules in complex systems, such as biological macromolecules, polymers, and non-biological systems. 

GROMACS was initially developed in the early 1990s at the University of Groningen in the Netherlands by the research group led by Herman Berendsen. The primary motivation was to create a software package for simulating molecular dynamics, particularly for biomolecular systems. The first versions of GROMACS were focused on biochemical molecules and were written in the Fortran programming language. The software aimed to efficiently simulate molecular systems using the GROMOS force field.

Ref: https://www.gromacs.org and https://www.gromacs.org/development.html

### Overview process of installation
- Get the latest version of your C and C++ compilers. (Minimum version: GNU (gcc/libstdc++) 9 or LLVM (clang/libc++) 7 or Microsoft (MSVC) 2019)
- Check that you have CMake version 3.18.4 or later.
- Get and unpack the latest version of the GROMACS tarball.
- Make a separate build directory and change to it.
- Run cmake with the path to the source as an argument
- Run make, and make install
- Source GMXRC to get access to GROMACS

### The System

OS System: CentOS Linux release 7.6.1810 with CMake 3.27.6, CUDA 11.8, GCC 11.2.1, OpenMPI 4.1.6, and PMIX 2.2.2 operated by Lmod \
Computer System: 2 x NVIDIA GeForce GTX 1070 Ti 8GB with Intel(R) Core(TM) i7-7700K CPU @ 4.20GHz

Refs: https://manual.gromacs.org/2024.2/install-guide/index.html

### Download GROMACS 2024.2 and Extract the tarball file
```
wget https://ftp.gromacs.org/gromacs/gromacs-2024.2.tar.gz
tar xfz gromacs-2024.2.tar.gz
```

### Compile and Install (with OpenMPI parallel and CUDA GPU)
```
cd gromacs-2024.2
mkdir build
cd build
cmake .. -DGMX_BUILD_OWN_FFTW=ON -DREGRESSIONTEST_DOWNLOAD=OFF -DGMX_MPI=on -DGMX_BUILD_MDRUN_ONLY=off -DGMX_GPU=CUDA -DGMX_SIMD=AVX2_256 -DCMAKE_INSTALL_PREFIX=/xxx/gromacs-2023.2
make install -j8
sudo make install
```

### Sample SLURM script for job submission
```
#!/bin/bash

################# Slurm Head #########################
 
# Set the number of nodes
#SBATCH --nodes=1

# Set the number of task(s)
#SBATCH --ntasks=1

# Set the number of task(s) per CPU core
#SBATCH --ntasks-per-core=1

# Set the number of GPU
#SBATCH --gres=gpu:1

# Set memory (RAM) for use
#SBATCH --mem=8GB

# Set wall time
#SBATCH --time=3-00:00:00

# Set partition
#SBATCH --partition=qgpu_gtx1070ti

# Set the name of a job
#SBATCH --job-name=gromacs_mdrun

######################################################

# Set module that is used in this script. Check by command 'ml av'
ml swap gnu7 gnu11
ml cmake/3.27.6 cuda/11.8 openmpi4 pmix/2.2.2

source /apps/gromacs-2024.2/bin/GMXRC

# Minimization
# In the case that there is a problem during minimization using a single precision of GROMACS, please try to use 
# a double precision of GROMACS only for the minimization step.
mpirun -np 1 gmx_mpi grompp -f step4.0_minimization.mdp -o step4.0_minimization.tpr -c step3_input.gro -r step3_input.gro -p topol.top -n index.ndx -maxwarn +50
mpirun -np 1 gmx_mpi mdrun -v -deffnm step4.0_minimization 

# Equilibration
mpirun -np 1 gmx_mpi grompp -f step4.1_equilibration.mdp -o step4.1_equilibration.tpr -c step4.0_minimization.gro -r step4.0_minimization.gro -p topol.top -n index.ndx -maxwarn +50
mpirun -np 1 gmx_mpi mdrun -v -deffnm step4.1_equilibration

# Production
mpirun -np 1 gmx_mpi grompp -f step5_production.mdp -o step5_production.tpr -c step4.1_equilibration.gro -p topol.top -n index.ndx -maxwarn +50
mpirun -np 1 gmx_mpi mdrun -v -deffnm step5_production

```
