### Installation essential modules/softwares
```
# yum -y install openmpi3-gnu7-ohpc mpich-gnu7-ohpc lmod-defaults-gnu7-openmpi3-ohpc
# yum -y install adios-gnu7-openmpi3-ohpc boost-gnu7-openmpi3-ohpc netcdf-gnu7-openmpi3-ohpc phdf5-gnu7-openmpi3-ohpc fftw-gnu7-openmpi3-ohpc hypre-gnu7-openmpi3-ohpc imp-gnu7-openmpi3-ohpc mfem-gnu7-openmpi3-ohpc  mpiP-gnu7-openmpi3-ohpc  mumps-gnu7-openmpi3-ohpc  netcdf-cxx-gnu7-openmpi3-ohpc netcdf-fortran-gnu7-openmpi3-ohpc netcdf-gnu7-openmpi3-ohpc petsc-gnu7-openmpi3-ohpc  pnetcdf-gnu7-openmpi3-ohpc scalapack-gnu7-openmpi3-ohpc scalasca-gnu7-openmpi3-ohpc scorep-gnu7-openmpi3-ohpc slepc-gnu7-openmpi3-ohpc superlu_dist-gnu7-openmpi3-ohpc tau-gnu7-openmpi3-ohpc trilinos-gnu7-openmpi3-ohpc singularity-ohpc hwloc-ohpc pmix-ohpc r-gnu7-ohpc hdf5-gnu7-ohpc mvapich2-gnu7-ohpc plasma-gnu7-ohpc scotch-gnu7-ohpc 
gnu8-compilers-ohpc intel-mpi-devel-ohpc openmpi3-gnu8-ohpc openmpi3-intel-ohpc python-mpi4py-gnu7-openmpi3-ohpc  python-mpi4py-gnu8-openmpi3-ohpc python-mpi4py-intel-openmpi3-ohpc python34-mpi4py-gnu7-openmpi3-ohpc python34-mpi4py-gnu8-openmpi3-ohpc  python34-mpi4py-intel-openmpi3-ohpc mpich-gnu8-ohpc  mpich-intel-ohpc 
# yum install --installroot=$CHROOT lmod-defaults-gnu7-openmpi3-ohpc
# yum -y install *gnu7-mpich* --exclude=lmod*
# yum -y install *intel-*-ohpc --exclude=lmod*
# yum -y install *-impi-* --exclude=lmod*
# yum -y install *python*-ohpc
```

### Copy lmod from compute node to master node
```
# cp -r /opt/ohpc/admin/images/centos7.5/opt/ohpc/admin/lmod/ /opt/ohpc/admin/
```

### Copy lmod.sh from $CHROOT/etc/profile.d to /etc/profile.d
```
# cp -r $CHROOT/etc/profile.d/lmod* /etc/profile.d
```

### Change vnfs.conf for copy locale folder to compute node by add "#"
```
# vi /etc/warewulf/vnfs.conf

#hybridize += /usr/share/locale
#hybridize += /usr/lib/locale
#hybridize += /usr/lib64/locale
```

### Share library to compute nodes
```
# vi $CHROOT/etc/fstab

192.168.1.254:/usr/lib /usr/lib nfs defaults 0 0
192.168.1.254:/usr/lib64 /usr/lib64 nfs defaults 0 0
192.168.1.254:/etc/slurm /etc/slurm nfs defaults 0 0
```
```
# vi /etc/exports 

#WWEXPORT:/usr/lib:192.168.1.0/255.255.255.0
/usr/lib 192.168.1.0/255.255.255.0(ro,no_root_squash)

#WWEXPORT:/usr/lib64:192.168.1.0/255.255.255.0
/usr/lib64 192.168.1.0/255.255.255.0(ro,no_root_squash)

#WWEXPORT:/etc/slurm:192.168.1.0/255.255.255.0
/etc/slurm 192.168.1.0/255.255.255.0(ro,no_root_squash)
```
```
# exportfs -a
# systemctl restart nfs-server
# systemctl enable nfs-server
```
