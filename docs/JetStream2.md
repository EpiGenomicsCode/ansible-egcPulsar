# JetStream2 Tips and Tricks
---
- Modifications needed for a JetStream2 VM to communicate with Galaxy

## Configure munge - Rocky 8

```
git clone https://github.com/dun/munge.git
cd munge
./bootstrap
./configure --prefix=/usr --sysconfdir=/etc --localstatedir=/var
make
make check
sudo make install
```

- Make mungekey
```
sudo -u munge mungekey --verbose
```

## Configure DRMAA - Ubuntu 22.04
- When using the SLURM Elastic cluster, SLURM DRMAA needs to be manually compiled and installed prior to running the Pulsar playbook

```
wget https://github.com/natefoo/slurm-drmaa/releases/download/1.1.3/slurm-drmaa-1.1.3.tar.gz
tar xzvf slurm-drmaa-1.1.3.tar.gz
cd slurm-drmaa-1.1.3/
sudo ./configure --with-slurm-inc=/usr/include/slurm
sudo make
sudo make install
```

## Modify default elastic node image to mount pulsar working directory
- The default elastic node image does not mount pulsar directory. A modified Git repo must be cloned and installed

1. Git clone the Elastic Cluster repo with Pulsar-specific changes
```
sudo su rocky
cd ~/
mkdir -p ORIG
mv CRI_Jetstream_Cluster/ ORIG/
git clone -b rocky-linux-pulsar https://github.com/WilliamKMLai/CRI_Jetstream_Cluster.git
```

2. Update the base image that will be propagated to elastic cluster images
- This requires re-running the CRI_Jetstream scripts

```
# Destroy the old image
./cluster_destroy_local.sh
# Turn off pulsar so it doesn’t compete with the head node
sudo systemctl disable pulsar
# Create the new image
./cluster_create_local.sh
# Turn pulsar back on
sudo systemctl enable pulsar
```

3. Update **/etc/exports** on head node to contain **/mnt/pulsar**, copy IP and permissions from previous entries

Example:
```
/home 10.0.62.157/24(rw,no_root_squash)
/export 10.0.62.157/24(rw,no_root_squash)
/opt/ohpc/pub 10.0.62.157/24(rw,no_root_squash)
# Add this here with proper internal IP address
/mnt/pulsar 10.0.62.157/24(rw,no_root_squash)
```

4. Restart the Pulsar VM headnode so the NFS server properly serves the /mnt/pulsar directory

## Debugging elastic node errors

By default, the elastic cluster is destroyed 30 seconds after the end of a detected job. On the head node, modify:
> /etc/slurm/slurm.conf

```
KillWait=30
```

Change 30 (seconds) to whatever time frame is necessary to investigate errors

The ssh key needed to login to the cluster node is located here:
> /etc/slurm/.ssh

Login to the cluster node:
```
ssh rocky@<internal IP>
```

## Restarting SLURM controller
The SLURM elastic controller doesn't always come back online after un-shelving an instance. You can check (and restart) the service using the following commands:

```
# Checking elastic cluster slurm service:
sudo service slurmctld status
```
```
# Restarting the service:
sudo service slurmctld restart
```

## Installing SLURM from scratch
From here: https://www.ni-sp.com/slurm-build-script-and-container-commercial-support/
Tested on Rocky 8

```
#
# Automatic SLURM built and installation script for EL7, EL8 and EL9 and derivatives
#
sudo yum install wget -y
wget --no-check-certificate https://www.ni-sp.com/wp-content/uploads/2019/10/SLURM_installation.sh
# set the desired version in case
export VER=20.11.9  #latest 20.11. VER=20.11.8
# export VER=21.08.5
# export VER=22.05.02  
bash SLURM_installation.sh
# wait a couple of minutes
# and test your SLURM installation yourself
sinfo
# see above for more SLURM commands and their output
```

SLURM config file located:
> /etc/slurm/slurm.conf

Updated to the following:
```
SlurmctldHost=localhost

MpiDefault=none
ProctrackType=proctrack/cgroup
ReturnToService=1
SlurmctldPidFile=/var/run/slurmctld.pid
SlurmdPidFile=/var/run/slurmd.pid
SlurmdSpoolDir=/var/spool/slurm/slurmd
SlurmUser=slurm
StateSaveLocation=/var/spool/slurm
SwitchType=switch/none
TaskPlugin=task/affinity

# SCHEDULING
SchedulerType=sched/backfill
SelectType=select/cons_res
SelectTypeParameters=CR_CPU_Memory

# LOGGING AND ACCOUNTING
AccountingStorageType=accounting_storage/none
ClusterName=cluster
JobAcctGatherType=jobacct_gather/none

# COMPUTE NODES
NodeName=localhost CPUs=32

# Partition
PartitionName=debug Default=YES Nodes=localhost State=UP
```
