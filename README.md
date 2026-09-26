<p align="center">
  <img src="docs/assets/cloudstack.svg" width="420" alt="Apache CloudStack">
</p>

# Apache Cloudstack Ansible Deployment

Apache CloudStack 4.22 automated with Ansible. Ansible deploys:
- Apache Cloudstack Management Server
- Apache Cloudstack Agent
- KVM Hypervisor (for the agents)
- NFS
- MySQL
- Ceph
### Requisites

Currently tested with Ubuntu Server 26.04 LTS. Works with ARM and x86 architectures.
### Repo Layout
```
.
├── ansible.cfg
├── docs
│   └── assets
│       └── cloudstack.svg
├── inventory
│   ├── group_vars
│   │   ├── all
│   │   │   ├── vars.yml
│   │   │   └── vault.yml
│   │   ├── ceph
│   │   │   ├── vars.yml
│   │   │   └── vault.yml
│   │   └── cloudstack_mgmt
│   │       ├── vars.yml
│   │       └── vault.yml
│   └── inventory.yml
├── LICENSE
├── main.yml
├── README.md
├── requirements.yml
└── roles
    ├── apache_mgmt
    │   ├── handlers
    │   │   └── main.yml
    │   └── tasks
    │       └── main.yml
    ├── ceph_cluster
    │   ├── defaults
    │   │   └── main.yml
    │   ├── tasks
    │   │   └── main.yml
    │   └── vars
    │       └── main.yml
    ├── ceph_prereqs
    │   ├── defaults
    │   │   └── main.yml
    │   ├── handlers
    │   │   └── main.yml
    │   ├── tasks
    │   │   └── main.yml
    │   └── templates
    │       └── chrony.conf.j2
    ├── common
    │   ├── handlers
    │   │   └── main.yml
    │   └── tasks
    │       └── main.yml
    ├── groups_quorum_check
    │   ├── defaults
    │   │   └── main.yml
    │   ├── files
    │   ├── handlers
    │   │   └── main.yml
    │   ├── meta
    │   │   └── main.yml
    │   ├── README.md
    │   ├── tasks
    │   │   └── main.yml
    │   ├── templates
    │   └── tests
    │       ├── inventory
    │       └── test.yml
    ├── kvm_init
    │   ├── defaults
    │   │   └── main.yml
    │   ├── handlers
    │   │   └── main.yml
    │   ├── tasks
    │   │   └── main.yml
    │   └── templates
    │       └── 01-netcfg.yaml.j2
    ├── nfs
    │   ├── defaults
    │   │   └── main.yml
    │   ├── handlers
    │   │   └── main.yml
    │   └── tasks
    │       └── main.yml
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

**NOTE:** You may overlap the same IP with seperate services. Some services probably should be only deployed once (such as the dashboards i.e. Ceph Dashboard and Apache Management. This is outlined in the inventory. However if you wanted to run KVM, CEPH OSD, Ceph Monitor, NFS, and Apache Management on the same server, that is permissible. The inventory should be viewed from a logical topology not a litteral of how many servers you are running either be one or five. It should deploy and function fine. Ceph is only recommended however if you have at least three servers, otherwise it's advisable to leave that blank and the script just will not deploy that service.

Modify the ansible user in ansible.cfg

`vim ansible.cfg`

Install nessecary collections

`ansible-galaxy collection install -r requirements.yml`

Run the playbook

`ansible-playbook main.yml --ask-vault-pass`
