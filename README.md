# ansible-egcPulsar

This playbooks is tested and working on JetStream2 and ROAR Collab.


## Requirements
- RHEL8
- Port 5671 open

### Install dependencies
> sudo yum -y install ansible git

### Prepare requirements for playbook
This playbook relies on an ansible-vault to store the keys required to communicate between Pulsar and Galaxy. This file is '.vault-password.txt' and should added to the playbook after cloning.

```
cd /storage/group/cfp102/default/
git clone -b roarcollab https://github.com/EpiGenomicsCode/ansible-egcPulsar.git
cd ansible-egcPulsar/pulsar
# Install ansible requirements
ansible-galaxy install -p roles -r requirements.yml
# Add .vault-password.txt
> cp .vault-password.txt /storage/group/cfp102/default/ansible-egcPulsar/pulsar
```

### Add slurm-drmaa compatibility
- Required for ROAR Collab
```
wget https://github.com/natefoo/slurm-drmaa/releases/download/1.1.4/slurm-drmaa-1.1.4.tar.gz
tar xzvf slurm-drmaa-1.1.4.tar.gz
cd slurm-drmaa-1.1.4/
export LD_LIBRARY_PATH=/storage/sys/slurm/lib
./configure  --with-slurm-inc=/storage/sys/slurm/include --with-slurm-lib=/storage/sys/slurm/lib --prefix=/mnt/pulsar/slurm-drmaa
make
# TODO Check to see if sudo is necessary
sudo make install
```

### Add Slurm compatibility
- For systems without a pre-existing slurm install (e.g., JS2)
- Uncomment:
```
    #- galaxyproject.repos
    #- galaxyproject.slurm
    ...
    #  post_tasks:
    #    - name: Install slurm-drmaa
    #      package:
    #        name: slurm-drmaa1
```

## Run playbook
sudo ansible-playbook pulsar.yml
