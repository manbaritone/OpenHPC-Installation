Credit: https://github.com/dasandata/Open_HPC/blob/master/Opertation%20Guide%20for%20OpenHPC%20Cluster.md

### 1. Add user
```
# wwuseradd testuser
# passwd testuser
# chage -d 0 testuser
# wwsh -y file import /etc/passwd
# wwsh -y file import /etc/group
# wwsh -y file import /etc/shadow
# wwsh -y provision set "*" --fileadd passwd,group,shadow
# wwsh file sync
```

### 2. Delete user
```
# userdel testuser
# rm -rf /home/testuser/
```

### 3. Install software on nodes

3.1 Install with direct to compute nodes
```
# pdsh -w node[1-2] 'yum install -y git'
```

3.2 Install to compute node image
```
# yum --installroot=/opt/ohpc/admin/images/centos7.5 install -y git
```

3.3 Mount compute node image
```
# wwvnfs --chroot /opt/ohpc/admin/images/centos7.5
```

### 4. Add NFS Mount on nodes
```
# cat /etc/exports
# echo "/DATA1 *(rw,no_subtree_check,no_root_squash)"  >>  /etc/exports
# exportfs -a
# pdsh -w node[1-2]  "df -hT | grep nfs"
# systemctl restart nfs-server.service
# cat /etc/fstab
# echo "master:/DATA1 /DATA1 nfs nfsvers=3 0 0"   >> /etc/fstab 
# wwvnfs --chroot /opt/ohpc/admin/images/centos7.5
# pdsh -w node[1-2]  "reboot"
# pdsh -w node[1-2]  "df -hT | grep nfs"
```

### 5. Module commands
```
# module --version
# ml --version
# module list
# ml list
# ml av
```

### 6. Install Python 3.5.4
```
# yum -y install zlib-devel bzip2-devel sqlite sqlite-devel openssl-devel libffi-devel
# cd /root
# PYTHON_VERSION=3.5.4
# wget https://www.python.org/ftp/python/${PYTHON_VERSION}/Python-${PYTHON_VERSION}.tgz
# tar  xvzf  Python-${PYTHON_VERSION}.tgz
# cd ~/Python-${PYTHON_VERSION}
# ./configure --enable-optimizations --with-ensurepip=install --enable-shared --prefix=/opt/ohpc/pub/apps/python3/${PYTHON_VERSION}
# make -j$(nproc) ; make install
```

### 7. Add Python Module
```
# cd /root
# PYTHON_VERSION=3.5.4
# git clone https://github.com/dasandata/Open_HPC
# cd /root/Open_HPC
# git pull
# cat /root/Open_HPC/Module_Template/python3.txt
# echo ${PYTHON_VERSION}
# mkdir /opt/ohpc/pub/modulefiles/python3
# cp -a /root/Open_HPC/Module_Template/python3.txt /opt/ohpc/pub/modulefiles/python3/${PYTHON_VERSION}
# sed -i "s/{VERSION}/${PYTHON_VERSION}/" /opt/ohpc/pub/modulefiles/python3/${PYTHON_VERSION}
# cat /opt/ohpc/pub/modulefiles/python3/${PYTHON_VERSION}
# rm -rf  ~/.lmod.d/.cache
# ml list
# ml av
# ml load python3/${PYTHON_VERSION}
# ml list
```

### 8. Add compute node
```
# wwsh -y node new c2 --ipaddr=192.168.1.253 --hwaddr=08:00:27:99:B3:4F -D enp0s8 
# wwsh node list
# wwsh -y provision set "c2" --vnfs=centos7.5 --bootstrap=`uname -r` --files=dynamic_hosts,passwd,group,shadow,network 
# systemctl restart gmond 
# systemctl restart gmetad 
# systemctl restart dhcpd 
# wwsh pxe update
```

```
# systemctl restart pbs
# wwsh file resync
# pdsh -w c2 uptime
# cd /etc/clustershell/groups.d
# vi local.cfg
compute: c[1-2]
```
