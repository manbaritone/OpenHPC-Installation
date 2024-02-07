### Install OpenHPC Repository
```
# yum -y install http://build.openhpc.community/OpenHPC:/1.3/CentOS_7/x86_64/ohpc-release-1.3-1.el7.x86_64.rpm
```

### Install basic package for OpenHPC
```
# yum -y install ohpc-base
# yum -y install ohpc-warewulf
# yum -y install chrony
```

### Setup Time Server
```
# systemctl enable ntpd.service
# systemctl enable chronyd
# echo "allow all" >> /etc/chrony.conf
# systemctl restart chronyd
```

### Install Slurm
```
# yum -y install ohpc-slurm-server
# perl -pi -e "s/ControlMachine=\S+/ControlMachine=master/" /etc/slurm/slurm.conf
```

### Determine enp0s3 for internal interface
```
# perl -pi -e "s/device = eth1/device = enp0s3/" /etc/warewulf/provision.conf
```

### Enable tftp service
```
# perl -pi -e "s/^\s+disable\s+= yes/ disable = no/" /etc/xinetd.d/tftp
```

### Determine ip for enp0s3
```
# ifconfig enp0s3 192.168.1.254 netmask 255.255.255.0 up
```

### Restart ans enable services
```
# systemctl restart xinetd
# systemctl enable mariadb.service # systemctl restart mariadb
# systemctl enable httpd.service
# systemctl restart httpd
# systemctl enable dhcpd.service
```

### Determine image for compute node
```
# export CHROOT=/opt/ohpc/admin/images/centos7.5
# wwmkchroot centos-7 $CHROOT
```

### Install OpenHPC for compute node
```
# yum -y --installroot=$CHROOT install ohpc-base-compute
```

### Copy resolv.conf to compute node
```
# cp -p /etc/resolv.conf $CHROOT/etc/resolv.conf
```

### Install slurm client in image compute node and master node
```
# yum -y --installroot=$CHROOT install ohpc-slurm-client
# yum -y install ohpc-slurm-client
```

### Add master host node to compute node
```
# vi $CHROOT/etc/hosts

master  192.168.1.254
```

### Install NTP for compute node
```
# yum -y --installroot=$CHROOT install ntp
# yum -y --installroot=$CHROOT install chrony
```

### Install kernel for compute node
```
# yum -y --installroot=$CHROOT install kernel
```

### Install modules user enviroment for compute node and master node
```
# yum -y --installroot=$CHROOT install lmod-ohpc
# yum -y install lmod-ohpc
```

### Create basic values for OpenHPC
```
# wwinit database 
# wwinit ssh_keys
```

### Create NFS Cilent 
(by mount /home and /opt/ohpc/pub from master node)
```
# echo "192.168.1.254:/home /home nfs nfsvers=3,nodev,nosuid,noatime 0 0" >> $CHROOT/etc/fstab 
# echo "192.168.1.254:/opt/ohpc/pub /opt/ohpc/pub nfs nfsvers=3,nodev,noatime 0 0" >> $CHROOT/etc/fstab
# echo "192.168.1.254:/share /share nfs nfsvers=3,nodev,nosuid,noatime 0 0" >> $CHROOT/etc/fstab 
```

### Create NFS Server
```
# echo "/home *(rw,no_subtree_check,fsid=10,no_root_squash)" >> /etc/exports 
# echo "/opt/ohpc/pub *(ro,no_subtree_check,fsid=11)" >> /etc/exports
# exportfs -a
# systemctl restart nfs-server
# systemctl enable nfs-server
```

### Determine kernel setting for compute node
```
# chroot $CHROOT systemctl enable ntpd
# chroot $CHROOT systemctl enable chrony
# echo "server 192.168.1.254 iburst" >> $CHROOT/etc/ntp.conf
# echo "server 192.168.1.254 iburst" >> $CHROOT/etc/chrony.conf
```

### Update basic slurm configuration
```
# perl -pi -e "s/^NodeName=(\S+)/NodeName=c[1-2]/" /etc/slurm/slurm.conf
# perl -pi -e "s/^PartitionName=normal Nodes=(\S+)/PartitionName=normal Nodes=c[1-2]/" /etc/slurm/slurm.conf
# perl -pi -e "s/^Sockets=(\S+)/Sockets=1/" /etc/slurm/slurm.conf
# perl -pi -e "s/^CoresPerSocket=(\S+)/CoresPerSocket=2/" /etc/slurm/slurm.conf
# perl -pi -e "s/^ThreadsPerCore=(\S+)/ThreadsPerCore=1/" /etc/slurm/slurm.conf
```

ClusterName=galaxy.cluster\
SlurmctldHost=master

```
# perl -pi -e "s/^NodeName=(\S+)/NodeName=c[1-2]/" $CHROOT/etc/slurm/slurm.conf
# perl -pi -e "s/^PartitionName=normal Nodes=(\S+)/PartitionName=normal Nodes=c[1-2]/" $CHROOT/etc/slurm/slurm.conf
# perl -pi -e "s/^Sockets=(\S+)/Sockets=1/" $CHROOT/etc/slurm/slurm.conf
# perl -pi -e "s/^CoresPerSocket=(\S+)/CoresPerSocket=2/" $CHROOT/etc/slurm/slurm.conf
# perl -pi -e "s/^ThreadsPerCore=(\S+)/ThreadsPerCore=1/" $CHROOT/etc/slurm/slurm.conf
```
ClusterName=galaxy.cluster\
SlurmctldHost=master

```
# systemctl enable munge
# systemctl enable slurmctld
# systemctl start munge
# systemctl start slurmctld
# chroot $CHROOT systemctl enable slurmd
```

### Determine memlock values
```
# perl -pi -e 's/# End of file/\* soft memlock unlimited\n$&/s' /etc/security/limits.conf
# perl -pi -e 's/# End of file/\* hard memlock unlimited\n$&/s' /etc/security/limits.conf
# perl -pi -e 's/# End of file/\* soft memlock unlimited\n$&/s' $CHROOT/etc/security/limits.conf 
# perl -pi -e 's/# End of file/\* hard memlock unlimited\n$&/s' $CHROOT/etc/security/limits.conf
```

### Determine rsyslog for compute node by point to master node
```
# perl -pi -e "s/\\#\\\$ModLoad imudp/\\\$ModLoad imudp/" /etc/rsyslog.conf
# perl -pi -e "s/\\#\\\$UDPServerRun 514/\\\$UDPServerRun 514/" /etc/rsyslog.conf 
# systemctl restart rsyslog
# echo "*.* @192.168.1.254:514" >> $CHROOT/etc/rsyslog.conf
# perl -pi -e "s/^\*\.info/\\#\*\.info/" $CHROOT/etc/rsyslog.conf
# perl -pi -e "s/^authpriv/\\#authpriv/" $CHROOT/etc/rsyslog.conf
# perl -pi -e "s/^mail/\\#mail/" $CHROOT/etc/rsyslog.conf
# perl -pi -e "s/^cron/\\#cron/" $CHROOT/etc/rsyslog.conf
# perl -pi -e "s/^uucp/\\#uucp/" $CHROOT/etc/rsyslog.conf
```

### Install Ganggila for monitoring OpenHPC
```
# yum -y install ohpc-ganglia
# yum -y --installroot=$CHROOT install ganglia-gmond-ohpc
# cp /opt/ohpc/pub/examples/ganglia/gmond.conf /etc/ganglia/gmond.conf 
# perl -pi -e "s/<sms>/master/" /etc/ganglia/gmond.conf
# cp /etc/ganglia/gmond.conf $CHROOT/etc/ganglia/gmond.conf 
# echo "gridname MySite.." >> /etc/ganglia/gmetad.conf
# systemctl enable gmond 
# systemctl enable gmetad 
# systemctl start gmond
# systemctl start gmetad
# chroot $CHROOT systemctl enable gmond 
# systemctl try-restart httpd
```

### Install Clustershell 
(by adm: master ansd compute:${compute_prefix}[1-${num_computes}] by compute_prefix = c and num_computes =2)
```
# yum -y install clustershell-ohpc
# cd /etc/clustershell/groups.d
# mv local.cfg local.cfg.orig
# echo "adm: master" > local.cfg
# echo "compute: c[1-2]" >> local.cfg
# echo "all: @adm,@compute" >> local.cfg
```

### Import files using in OpenHPC
```
# wwsh file import /etc/passwd 
# wwsh file import /etc/group
# wwsh file import /etc/shadow
# wwsh file import /etc/slurm/slurm.conf
# wwsh file import /etc/munge/munge.key
# wwsh file list
```

### Determine bootstrap image
```
# export WW_CONF=/etc/warewulf/bootstrap.conf 
# echo "drivers += updates/kernel/" >> $WW_CONF 
# echo "drivers += overlay" >> $WW_CONF
```

### Setup bootstrap image
```
# wwbootstrap `uname -r`
```

### Create Virtual Node File System (VNFS) image
```
# wwvnfs --chroot $CHROOT
```

### Determine compute node by MAC Address 
(by GATEWAYDEV=enp0s8 is Public interface // using in internal lan group)
```
# echo "GATEWAYDEV=enp0s8" > /tmp/network.$$
# wwsh -y file import /tmp/network.$$ --name network
# wwsh -y file set network --path /etc/sysconfig/network --mode=0644 --uid=0 
# wwsh file list
```

We have 2 compute nodes
```
# wwsh -y node new c1 --ipaddr=192.168.1.253 --hwaddr=08:00:27:99:B3:4F -D enp0s8 
# wwsh -y node new c2 --ipaddr=192.168.1.252 --hwaddr=08:00:27:99:B3:5F -D enp0s8 
# wwsh node list
```

### Determine VNFS for compute node
```
# wwsh -y provision set "c1" --vnfs=centos7.5 --bootstrap=`uname -r` --files=dynamic_hosts,passwd,group,shadow,slurm.conf,munge.key,network
# wwsh -y provision set "c2" --vnfs=centos7.5 --bootstrap=`uname -r` --files=dynamic_hosts,passwd,group,shadow,slurm.conf,munge.key,network
# wwsh provision list
```

### Restart ganglia services
```
# systemctl restart gmond 
# systemctl restart gmetad 
# systemctl restart dhcpd 
# wwsh pxe update
```

### Sync all file to compute node
```
# wwsh file resync
```

### Compute node installation (by booting from network with MAC Address)
Open compute node and waiting for finish and you can check by:
```
# pdsh -w c1 uptime
# pdsh -w c[1-2] uptime
```

### Resource Manager Startup
```
# systemctl restart munge
# systemctl restart slurmctld
# pdsh -w c[1-2] systemctl restart slurmd
```

### Test mung by
```
# munge -n | unmunge
# munge -n | ssh c1 unmunge
# munge -n | ssh c2 unmunge
```

### Test slurm by
```
# systemctl status slurmctld
# ssh c1 systemctl status slurmd
# ssh c2 systemctl status slurmd
```

### Test resource by
```
# scontrol show nodes
```
