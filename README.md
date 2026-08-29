<p align="center">
  <img src="docs/assets/cloudstack.svg" width="420" alt="Apache CloudStack">
</p>

# Apache Cloudstack Ansible Deployment

Apache CloudStack 4.22 automated with Ansible. Ansible deploys:
- Apache Cloudstack Management Server
- Apache Cloudstack Agent
- KVM Hypervisor (for the agents)
- NFS
### Requisites

Currently tested with Ubuntu Server 26.04 LTS. Works with ARM and x86 architectures.
### Repo Layout
```
.
├── ansible.cfg  
├── inventory  
│   ├── group_vars  
│   │   └── all  
│   │       └── vault.yml.example  
│   └── inventory.yml  
├── LICENSE  
├── main.yml  
├── playbooks  
│   ├── ceph.yml  
│   ├── kvm.yml  
│   ├── mgmt_server.yml  
│   └── nfs.yml  
├── README.md  
├── requirements.yml  
└── roles  
   ├── apache_mgmt  
   │   ├── handlers  
   │   │   └── main.yml  
   │   └── tasks  
   │       └── main.yml  
   ├── common  
   │   └── tasks  
   │       └── main.yml  
   ├── kvm_init  
   │   ├── handlers  
   │   │   └── main.yml  
   │   ├── tasks  
   │   │   └── main.yml  
   │   └── templates  
   │       └── 01-netcfg.yaml.j2  
   ├── nfs  
   │   ├── handlers  
   │   │   └── main.yml  
   │   └── tasks  
   │       └── main.yml  
   └── repo_init  
       └── tasks  
           └── main.yml
```
### Instructions

First pull the repo down.

`git clone https://github.com/DRivera-UL/Ansible-Cloudstack-Deployment`

Encrypt the vault.yml secret.

`ansible-vault encrypt ./Ansible-Cloudstack-Deployment/inventory/group_vars/all/vault.yml`

Change the passwords

`ansible-vault edit vault.yml`

Edit the inventory file. All services are agnostic and *may* be ran on seperate devices or all on once device.

The only recommendation I will make is to not deploy the apache management server to multiple devices as the script is not designed to loadbalance MySQL servers *at the moment* however feel free to expirement with multiple KVMs or NFS servers as you wish.

Modify the inventory file

`vim inventory/inventory.yml`

Modify the ansible user in ansible.cfg

`vim ansible.cfg`

Install nessecary collections

`ansible-galaxy collection install -r requirements.yml`

Run the playbook

`ansible-playbook main.yml --ask-vault-pass`
