# ansible-egcPulsar

This playbooks is tested and working on JetStream2 and ROAR Collab.

<details>
<summary>Development notes</summary>
- This playbook assumes a ROAR Collab service account already exists on the system. If running this playbook for development purposes, this user will need to be added to the system before running the playbooks

```
sudo adduser other_5f6ad95074eb4e
```
</details>

### Requirements
- RHEL8
- Port 5671 open

### Install dependencies
```
sudo yum -y install ansible git
```

### Prepare requirements for playbook
This playbook relies on an ansible-vault to store the keys required to communicate between Pulsar and Galaxy. This file is '.vault-password.txt' and should added to the playbook after cloning.

- sudo as service account 'other_5f6ad95074eb4e'

```
cd /storage/home/other_5f6ad95074eb4e/
git clone -b roarcollab https://github.com/EpiGenomicsCode/ansible-egcPulsar.git
# Add .vault-password.txt
cp /storage/home/other_5f6ad95074eb4e/.vault-password.txt ansible-egcPulsar/pulsar
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

### Run playbook
```
sudo ansible-playbook pulsar.yml
```

## Add genome-builds
- ROAR Collab does not support CVMFS for all nodes and EGC customers often require additional rare genome builds outside CVMFS support
- Mirroring the genome build structure from the host Galaxy instance on the Pulsar remote server allows for 'local' genome build alignment through a Pulsar server
- **Note** The current implementation is VERY brittle and will not scale well across multiple Pulsar nodes

```
cd /storage/group/bfp2/default/00_pughlab/
wget <URL HERE>
tar xzvf <GENOME FOLDER>
```
