# ansible-Pulsar

## Tested on RHEL8
## Need ansible installed
> sudo yum -y install ansible
## Need apptainer installed
> sudo yum -y apptainer

## Install requirements for playbook
> cd /storage/group/cfp102/default/ansible-egcPulsar/pulsar
>
> ansible-galaxy install -p roles -r requirements.yml
>
> ansible-galaxy role install galaxyproject.cvmfsexec

## Add .vault-password.txt
> cp .vault-password.txt /storage/group/cfp102/default/ansible-egcPulsar/pulsar

## Add slurm-drmaa compatibility
> wget https://github.com/natefoo/slurm-drmaa/releases/download/1.1.4/slurm-drmaa-1.1.4.tar.gz
>
> tar xzvf slurm-drmaa-1.1.4.tar.gz
>
> cd slurm-drmaa-1.1.4/
>
> export LD_LIBRARY_PATH=/storage/sys/slurm/lib
>
> ./configure  --with-slurm-inc=/storage/sys/slurm/include --with-slurm-lib=/storage/sys/slurm/lib --prefix=/mnt/pulsar/slurm-drmaa
>
> make
>
//Check to see if sudo is necessary
> sudo make install


## Run playbook
sudo ansible-playbook pulsar.yml
