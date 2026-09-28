Ceph
=========

It's a "turn-key" role to deploy Ceph and let cephadm dictacte mon and mgr placement while pulling all available devices as osd devices for all devices specified in the inventory.

Requirements
------------

Ansible Galaxy: ceph.automation

Role Variables
--------------

Variables in the defaults/main.yml:

`ceph_set_hostname:`

This allows the role to change the hostnames to the ansible inventory hostname so as the hostnames being static and unique is a depenacy for Ceph to map the CRUSH map properly

`ceph_group:`

This allows the role to understand which inventory group it's apart of. This helps to not hardcode the inventory name in the playbook as some inventory values are used nested inside of tasks so simpily importing a task to a group is not enough since the listed items communicate with eachother.

`ceph_bootstrap_host:`

Defines which ansible host is the bootstrap server. Applied a default value to select the first host in the group from the list but free to select another bootstrap host.

`ceph_image:`

Can define the Ceph container image for pinning the container image. Otherwise the package will define the container image if left NULL.

`ceph_dashboard_user:`

Username for the Ceph dashboard webUI.

`ceph_dashboard_password:`

Password for the Ceph dashboard webUI.

```
ceph_service_specs:
  - service_type: mon
    placement:
      count: 3
  - service_type: mgr
    placement:
      count: 2
  - service_type: osd
    service_id: all_available
    placement:
      host_pattern: "*"
    spec:
      data_devices:
        all: true
```

The listed variable will tell cephadm how to deploy the cluster. Most cluster will require a mon count of 3, however 5 may be needed for medium clusters and 7 for the larger cluster. The more mons you have the slower your cluster will run so be mindful. Also, this playbook requires quorum so the **mon count MUST be an odd number** otherwise the role will assert a failure. Quorom is important to prevent split brain conditions. 

```
pools:
  - name: cephpool
    size: 3
    application: rbd # Or set it to "rgw" or "cephfs"
    pool_type: replicated
    pg_autoscale_mode: "on"
```

Use this variable list to define the pool name and specifications of the pool made inside of the cluster.

`ceph_pool_user:`

Creates a user and generates a key. This role, *even though it does not need it,* registers `ceph_key_secret` as a variable to be used in broader playbooks that require the account to connect to pool.

Variables in the vars/main.yml

Used this YAML to parse and output the nested variables in defaults/main.yml.

`ceph_mon_spec:`

Prints the mon number from `ceph_service_spec:` inside defaults/main.yml. The number is used to check and assert the mon count is equal or less than the inventory group size and is an odd number.

`ceph_pool_name:`

Prints the pool name as a string from `pools` in defaults/main.yml. Needed to create the user to the newly made pool.

Dependencies
------------

Currently needs Ubunutu 26.04 since the role does not use Ansible gather facts for release version specific plays since the repo would need to be added for that.

Example Playbook
----------------

Including an example of how to use your role (for instance, with variables passed in as parameters) is always nice for users too:

    - hosts: ceph
      roles:
        - role: ./roles/ceph
          become: true

License
-------

Apache 2.0

Author Information
------------------

Dante Rivera
