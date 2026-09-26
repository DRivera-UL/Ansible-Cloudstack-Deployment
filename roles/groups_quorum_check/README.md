Groups Quorum Check
=========

Checks if an inventory item has an even or odd number of items. Good to use for things that should have an odd number such as SQL server replication, Kubernetes control planes, or Ceph monitors for example.

Requirements
------------

Any pre-requisites that may not be covered by Ansible itself or the role should be mentioned here. For instance, if the role uses the EC2 module, it may be a good idea to mention in this section that the boto package is required.

Role Variables
--------------

The defaults/main.yml will have a variable called "quorum_groups" where you may define your inventory file groups that must be quorum. 

Also defaults/main.yml will include "quorum_strict" where the task will ignore the failure to serve as a warning since quorem may not strictly cause
quorem items to fail but is best practice and can be deployed non-quorem if scaling is accounted for but still want to deploy to the hosts.

Dependencies
------------

A list of other roles hosted on Galaxy should go here, plus any details in regards to parameters that may need to be set for other roles, or variables that are used from other roles.

Example Playbook
----------------

Including an example of how to use your role (for instance, with variables passed in as parameters) is always nice for users too:

    - hosts: servers
      roles:
         - groups_quorum_check

License
-------

MIT

Author Information
------------------

Dante Rivera
