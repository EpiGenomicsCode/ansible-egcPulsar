# JetStream2 Tips and Tricks
---
- Modifications needed for a JetStream2 VM Elastic Cluster to communicate with Galaxy

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
The SLURM controller doesn't always come back online after unshelving an instance. You can check (and restart) the service using the following commands:

```
# Checking elastic cluster slurm service:
sudo service slurmctld status
```
```
# Restarting the service:
sudo service slurmctld restart
```
