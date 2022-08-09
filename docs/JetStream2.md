# JetStream2 Tips and Tricks
---
- Modifications needed for a JetStream2 VM to communicate with Galaxy

## Configure DRMAA
- When using the SLURM Elastic cluster, SLURM DRMAA needs to be manually compiled and installed prior to running the Pulsar playbook

```
wget https://github.com/natefoo/slurm-drmaa/releases/download/1.1.3/slurm-drmaa-1.1.3.tar.gz
tar xzvf slurm-drmaa-1.1.3.tar.gz
cd slurm-drmaa-1.1.3/
sudo ./configure --with-slurm-inc=/usr/include/slurm
sudo make
sudo make install
```

## Update Elastic node image
- Changes to the base image that need to be propagated to the elastic cluster image require re-running the CRI_Jetstream scripts

```
sudo su - rocky
cd CRI_Jetstream_Cluster/
# Destroy the old image
./cluster_destroy_local.sh
# Turn off pulsar so it doesn’t compete with the head node
sudo systemctl disable pulsar
# Create the new image
./cluster_create_local.sh
```

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
