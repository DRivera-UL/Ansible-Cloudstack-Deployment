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
## Instructions

### Preperation

First pull the repo down.

`git clone https://github.com/DRivera-UL/Ansible-Cloudstack-Deployment`

`cd Ansible-Cloudstack-Deployment`

Batched edit and then encrypt the **all** vault.yml secret.

### Initial Password Managment

`vim -p inventory/group_vars/*/vault.yml`

`ansible-vault encrypt inventory/group_vars/*/vault.yml`

### Inventory Management

Edit the inventory file.

`vim -p inventory/inventory.yml`

*example inventory file*
```
   1 │ ---
   2 │ all:
   3 │   hosts:
   4 │     node01: { ansible_host: 192.168.1.3 }
   5 │     node02: { ansible_host: 192.168.1.4 }
   6 │     node03: { ansible_host: 192.168.1.5 }
   7 │   children:
   8 │     cloudstack_mgmt:
   9 │       hosts:
  10 │         node01:
  11 │     kvm_host:
  12 │       hosts:
  13 │         node01:
  14 │         node02:
  15 │         node03:
  16 │     nfs_server:
  17 │       hosts:
  18 │         node01:
  19 │     ceph:
  20 │       hosts:
  21 │         node01:
  22 │         node02:
  23 │         node03:
```

#### All hosts in cluster instructions

Under all hosts please dicated the hostname and IP of the remote system. This playbook will modify the hostname of the system as Ceph requires a unique hostname for the CRUSH map. In the example above "node01", "node02", and "node03" will be the hostname that is pushed to the respective remote systems 192.168.1.3-5. You should not change the hostname later as Ceph design is dependant on the hostname.

#### Cloudstack Management Group

Now with the newly defined hostnames under the "cloudstack_mgmt" group under "hosts" import the hostname followed by a ":". This playbook is designed to have one management server at the moment since the SQL backend is tied to that role. Please assign one device to serve as the managment service (front end web UI and SQL backend) for the Cloudstack management cluster.

#### KVM Host Group

This is where the VMs will be hosted using KVM. You are free to have this deploy to the management server as well if you wish to have a server manage and virtualize to itself rather than a dedicated managment server. This is not unusual for small clusters. For example, node01 can server as a Ceph OSD, NFS server, KVM host and CloudStack management host if you wish.

#### NFS Server Group

If you are not bringing your own storage solution pick the servers that will serve your secondary storage (needed to ISOs, QCOW2 templates, etc.) for the cluster. NFS can also serve as the primary storage too and the defaults will deploy two NFS exports "(IP of hosts)/exports/primary" and "(IP of hosts)/exports/secondary".

#### Ceph Group

The design in this playbook is to let Cephadm determine which hosts will run the Admin, Mgr, and Mon but ***all hosts here will pull all available devices (any unused disk) into the Ceph crush map***. Ideally you have at least three identical server with drives of the same size and quantity on each server, but most importantly each server minimally is running the same blank drive space on each one. If you have a lopsided server with more drives than another *Ceph will accept it* but the storage pool might not utilize all drives added depending on the replication settings and the storage space of the smallest server. For simplicity, please try to provide servers with the same unallocated storage size and per Ceph recommendations at least three servers for block storage.

Modify the ansible user in ansible.cfg

`vim ansible.cfg`

Install nessecary collections

`ansible-galaxy collection install -r requirements.yml`

Run the playbook

`ansible-playbook main.yml --ask-vault-pass`
