Credit: https://github.com/dasandata/Open_HPC/blob/master/Provisioning/GPU%20Node%20Provisioning%20of%20OpenHPC%20Cluster.md 

### Create and source ohpc_variable.sh
```
# vi ohpc_variable.sh

export CHROOT=/opt/ohpc/admin/images/centos7.5
```

### Install CUDA repository on master node
```
# curl -L -o cuda-repo-rhel7-10.0.130-1.x86_64.rpm http://developer.download.nvidia.com/compute/cuda/repos/rhel7/x86_64/cuda-repo-rhel7-10.0.130-1.x86_64.rpm
# yum -y install cuda-repo-rhel7-10.0.130-1.x86_64.rpm
```

### Copy epel-nvidia.repo to compute node
```
# yum -y install --installroot ${CHROOT} cuda-repo-rhel7-10.0.130-1.x86_64.rpm
```

### Install libGLU.so libX11.so libXi.so libXmu.so to master node
```
# yum -y install libXi-devel mesa-libGLU-devel libXmu-devel libX11-devel freeglut-devel libXm* openmotif*
```

### Install libGLU.so libX11.so libXi.so libXmu.so to compute node
```
# yum -y install --installroot=$CHROOT libXi-devel mesa-libGLU-devel libXmu-devel libX11-devel freeglut-devel libXm* openmotif*
```

### Install nvidia-driver and cuda toolkit to master node 
```
# yum -y install nvidia-driver nvidia-settings cuda nvidia-driver-cuda
```

### Install nvidia-driver and cuda toolkit to compute node 
```
# yum -y install --installroot=$CHROOT nvidia-driver nvidia-settings cuda nvidia-driver-cuda
```

### Install cuda-cudnn
First register here: https://developer.nvidia.com/developer-program/signup and Download cuda-cudnn tar.xz files from https://developer.nvidia.com/rdp/cudnn-archive
```
# tar -xvf cudnn-linux-x86_64-*****.tar.xz
# mv cudnn-linux-x86_64-***** cuda
# cp -P cuda/include/cudnn.h /usr/local/cuda-11.8/include
# cp -P cuda/lib/libcudnn* /usr/local/cuda-11.8/lib64/
# chmod a+r /usr/local/cuda-11.8/lib64/libcudnn*
```
Nvidia device enable on boot (/dev/nvidia*)

```
# chroot $CHROOT
```

```
# vi /etc/init.d/nvidia

#!/bin/bash
#
# nvidia    Set up NVIDIA GPU Compute Accelerators
#
# chkconfig: 2345 55 25
# description:    NVIDIA GPUs provide additional compute capability. \
#    This service sets the GPUs into the desired state.
#
# config: /etc/sysconfig/nvidia

### BEGIN INIT INFO
# Provides: nvidia
# Required-Start: $local_fs $network $syslog
# Required-Stop: $local_fs $syslog
# Should-Start: $syslog
# Should-Stop: $network $syslog
# Default-Start: 2 3 4 5
# Default-Stop: 0 1 6
# Short-Description: Set GPUs into the desired state
# Description:    NVIDIA GPUs provide additional compute capability.
#    This service sets the GPUs into the desired state.
### END INIT INFO


################################################################################
######################## Microway Cluster Management Software (MCMS) for OpenHPC
################################################################################
#
# Copyright (c) 2015-2016 by Microway, Inc.
#
# This file is part of Microway Cluster Management Software (MCMS) for OpenHPC.
#
#    MCMS for OpenHPC is free software: you can redistribute it and/or modify
#    it under the terms of the GNU General Public License as published by
#    the Free Software Foundation, either version 3 of the License, or
#    (at your option) any later version.
#
#    MCMS for OpenHPC is distributed in the hope that it will be useful,
#    but WITHOUT ANY WARRANTY; without even the implied warranty of
#    MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
#    GNU General Public License for more details.
#
#    You should have received a copy of the GNU General Public License
#    along with MCMS.  If not, see <http://www.gnu.org/licenses/>
#
################################################################################


# source function library
. /etc/rc.d/init.d/functions

# Some definitions to make the below more readable
NVSMI=/usr/bin/nvidia-smi
NVCONFIG=/etc/sysconfig/nvidia
prog="nvidia"

# default settings
NVIDIA_ACCOUNTING=1
NVIDIA_PERSISTENCE_MODE=0
NVIDIA_COMPUTE_MODE=1
NVIDIA_CLOCK_SPEEDS=max
# pull in sysconfig settings
[ -f $NVCONFIG ] && . $NVCONFIG

RETVAL=0


# Determine the maximum graphics and memory clock speeds for each GPU.
# Create an array of clock speed pairs (memory,graphics) to be passed to nvidia-smi
declare -a MAX_CLOCK_SPEEDS
get_max_clocks()
{
    GPU_QUERY="$NVSMI --query-gpu=clocks.max.memory,clocks.max.graphics --format=csv,noheader,nounits"

    MAX_CLOCK_SPEEDS=( $($GPU_QUERY | awk '{print $1 $2}') )
}


start()
{
    /sbin/lspci | grep -qi nvidia
    if [ $? -ne 0 ] ; then
        echo -n $"No NVIDIA GPUs present. Skipping NVIDIA GPU tuning."
        warning
        echo
        exit 0
    fi

    echo -n $"Starting $prog: "

    # If the nvidia-smi utility is missing, this script can't do its job
    [ -x $NVSMI ] || exit 5

    # A configuration file is not required
    if [ ! -f $NVCONFIG ] ; then
        echo -n $"No GPU config file present ($NVCONFIG) - using defaults"
        echo
    fi

    # Set persistence mode first to speed things up
    echo -n "persistence"
    $NVSMI --persistence-mode=$NVIDIA_PERSISTENCE_MODE 1> /dev/null
    RETVAL=$?

    if [ ! $RETVAL -gt 0 ]; then
        echo -n " accounting"
        $NVSMI --accounting-mode=$NVIDIA_ACCOUNTING 1> /dev/null
        RETVAL=$?
    fi

    if [ ! $RETVAL -gt 0 ]; then
        echo -n " compute"
        $NVSMI --compute-mode=$NVIDIA_COMPUTE_MODE 1> /dev/null
        RETVAL=$?
    fi


    if [ ! $RETVAL -gt 0 ]; then
        echo -n " clocks"
        if [ -n "$NVIDIA_CLOCK_SPEEDS" ]; then
            # If the requested clock speed value is "max",
            # work through each GPU and set to max speed.
            if [ "$NVIDIA_CLOCK_SPEEDS" == "max" ]; then
                get_max_clocks

                GPU_COUNTER=0
                GPUS_SKIPPED=0
                while [ "$GPU_COUNTER" -lt ${#MAX_CLOCK_SPEEDS[*]} ] && [ ! $RETVAL -gt 0 ]; do
                    if [[ ${MAX_CLOCK_SPEEDS[$GPU_COUNTER]} =~ Supported ]] ; then
                        if [ $GPUS_SKIPPED -eq 0 ] ; then
                            echo
                            GPUS_SKIPPED=1
                        fi
                        echo "Skipping non-boostable GPU"
                    else
                        $NVSMI -i $GPU_COUNTER --applications-clocks=${MAX_CLOCK_SPEEDS[$GPU_COUNTER]} 1> /dev/null
                    fi
                    RETVAL=$?

                    GPU_COUNTER=$(( $GPU_COUNTER + 1 ))
                done
            else
                # This sets all GPUs to the same clock speeds (which only works
                # if the GPUs in this system are all the same).
                $NVSMI --applications-clocks=$NVIDIA_CLOCK_SPEEDS 1> /dev/null
            fi
        else
            $NVSMI --reset-applications-clocks 1> /dev/null
        fi
        RETVAL=$?
    fi

    if [ ! $RETVAL -gt 0 ]; then
        if [ -n "$NVIDIA_POWER_LIMIT" ]; then
            echo -n " power-limit"
            $NVSMI --power-limit=$NVIDIA_POWER_LIMIT 1> /dev/null
            RETVAL=$?
        fi
    fi

    if [ ! $RETVAL -gt 0 ]; then
        success
    else
        failure
    fi
    echo
    return $RETVAL
}

stop()
{
    /sbin/lspci | grep -qi nvidia
    if [ $? -ne 0 ] ; then
        echo -n $"No NVIDIA GPUs present. Skipping NVIDIA GPU tuning."
        warning
        echo
        exit 0
    fi

    echo -n $"Stopping $prog: "
    [ -x $NVSMI ] || exit 5

    $NVSMI --persistence-mode=0 1> /dev/null && success || failure
    RETVAL=$?
    echo
    return $RETVAL
}

restart() {
    stop
    start
}

force_reload() {
    restart
}

status() {
    $NVSMI
}

case "$1" in
    start)
        start
        ;;
    stop)
        stop
        ;;
    restart)
        restart
        ;;
    force-reload)
        force_reload
        ;;
    status)
        status
        RETVAL=$?
        ;;
    *)
        echo $"Usage: $0 {start|stop|restart|force-reload|status}"
        RETVAL=2
esac
exit $RETVAL
```

```
# chmod +x /etc/init.d/nvidia
# exit
```

```
# vi /lib/systemd/system/nvidia-gpu.service

[Unit]
Description=NVIDIA GPU Initialization
After=remote-fs.target

[Service]
Type=oneshot
RemainAfterExit=yes
ExecStart=/etc/init.d/nvidia start
ExecStop=/etc/init.d/nvidia stop

[Install]
WantedBy=multi-user.target
```

```
# chroot $CHROOT
# systemctl enable nvidia-gpu.service
```

### Update to Node nvfs image
```
# wwvnfs --chroot ${CHROOT}
# wwsh vnfs list
```

### Apply update imgae to nodes (rebooting)
```
# ssh node1 reboot
```

### Download and install cuda-8.0, cuda-9.0, cuda-10.0 toolkits
```
# cd /root
# wget https://developer.nvidia.com/compute/cuda/8.0/Prod2/local_installers/cuda_8.0.61_375.26_linux-run
# mv cuda_8.0.61_375.26_linux-run cuda_8.0.61_375.26_linux.run
# wget https://developer.nvidia.com/compute/cuda/9.0/Prod/local_installers/cuda_9.0.176_384.81_linux-run
# mv cuda_9.0.176_384.81_linux-run cuda_9.0.176_384.81_linux.run
# wget https://developer.nvidia.com/compute/cuda/10.0/Prod/local_installers/cuda_10.0.130_410.48_linux
# mv cuda_10.0.130_410.48_linux cuda_10.0.130_410.48_linux.run
# ./cuda_8.0.61_375.26_linux.run --silent --toolkit
# ./cuda_9.0.176_384.81_linux.run --silent --toolkit
# ./cuda_10.0.130_410.48.linux.run --silent --toolkit
```

### Add Multiple Cuda Module for GPU Node
```
# cd /root
# git clone https://github.com/dasandata/open_hpc
# cd /root/open_hpc
# git pull
# mkdir -p /opt/ohpc/pub/modulefiles/cuda
# cd
```

### Add CUDA Module File by each version
```
# export GIT_CLONE_DIR=/root/open_hpc
# export MODULES_DIR=/opt/ohpc/pub/modulefiles
# for CUDA_VERSION in 8.0 9.0 10.0 ; do
cp -a ${GIT_CLONE_DIR}/Module_Template/cuda.lua ${MODULES_DIR}/cuda/${CUDA_VERSION}.lua ;
sed -i "s/{version}/${CUDA_VERSION}/" ${MODULES_DIR}/cuda/${CUDA_VERSION}.lua ;
done
```

### Refresh modules
```
# rm -rf  ~/.lmod.d/.cache
# module av
```
