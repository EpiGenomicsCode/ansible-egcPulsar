# Starting point
---
- This guide assumes a clean build of Rocky 8 that you have sudo rights on.

# Getting started
---
[ansible](https://en.wikipedia.org/wiki/Ansible_(software)) will install all required dependencies on the target server, so this section primarily refers to the dependencies required to run ansible from your local workstation.

## Install dependencies


#### Rocky 8 instructions


## Install dependencies on local workstation
- Recommended to use ansible version >=2.7

#### Rocky 8 instructions

##### Install Ansible
python3 -m pip install --user ansible

#### MacOS instructions
This assumes you are managing software packages through [homebrew](https://brew.sh/)

##### ansible Install
```
brew install ansible
```
##### sshpass Install
- This is required for passing the sudo password to the provisioned machine. Note that Homebrew officially does NOT support the use of sshpass. This command pulls sshpass from a 3rd party source and compiles it locally. Use at your own risk.
- Generally you should be using SSH keys instead of passing your password over the internet.
```
brew install hudochenkov/sshpass/sshpass
```

#### Ubuntu instructions
```
sudo apt update
sudo apt install software-properties-common
sudo apt-add-repository --yes --update ppa:ansible/ansible
sudo apt install ansible
```

```
sudo apt install sshpass -y
```

## Deploy production Pulsar

Download roles for ansible
```
cd pulsar/
ansible-galaxy install -r requirements.yml
```

Deploy ansible playbook
- -k allows for passing of SSH password through commandline
- -K allows for passing of sudo password through commandline

```
ansible-playbook -kK pulsar.yml
```
