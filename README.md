# ansible-Pulsar

## Tested on RHEL8
## Need ansible installed
> sudo yum -y install ansible

## Install requirements for playbook
> cd /storage/group/cfp102/default/ansible-egcPulsar/pulsar
> ansible-galaxy install -p roles -r requirements.yml

## Add .vault-password.txt to
### /storage/group/cfp102/default/ansible-egcPulsar/pulsar

## Run playbook
sudo ansible-playbook pulsar.yml
