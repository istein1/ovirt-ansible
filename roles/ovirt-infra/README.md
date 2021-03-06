oVirt infra
=========

This role setup oVirt infrastructure including: data centers, clusters, networks, hosts, users and groups.

Requirements
------------

 * oVirt Python SDK version 4
 * Ansible version 2.3

Role Variables
--------------

From [ovirt-datacenters]:

| Name                                | Default value         |                                      |
|-------------------------------------|-----------------------|--------------------------------------|
| data_center_name                    | UNDEF                 | Name of the data center              |
| data_center_description             | UNDEF                 | Description of the data center       |
| data_center_local                   | false                 | Whether the data center should be shared or local |
| compatibility_version               | UNDEF                 | Compatibility version of data center |

From [ovirt-clusters]:

This role accept only one variable called `clusters`. To see example of this variable please check the role's documentation.

From [ovirt-hosts]:

This role accept only one variable called `hosts`. To see example of this variable please check the role's documentation.

From [ovirt-networks]:

This role accept variable called `logical_networks`, which setup logical networks in oVirt. To see example of this variable please check the role's documentation.
This role accept variable called `host_networks`, which setup host networks in oVirt. To see example of this variable please check the role's documentation.

From [ovirt-storages]:

This role accept variable called `storages`, which setup different storages in oVirt. To see example of this variable please check the role's documentation.

From [ovirt-aaa-jdbc]:

This role accept variable called `users` and `groups`, which creates users and groups in AAA jdbc extension. To see example of this variable please check the role's documentation.

From [ovirt-permissions]:

This role accept variable called `permissions`, which manges permissions of users and groups. To see example of this variable please check the role's documentation.

Dependencies
------------

 * [ovirt-datacenters]
 * [ovirt-clusters]
 * [ovirt-hosts]
 * [ovirt-networks]
 * [ovirt-storages]
 * [ovirt-aaa-jdbc]
 * [ovirt-permissions]

Example Playbook
----------------

```yaml
---
- name: oVirt infra
  hosts: localhost
  connection: local
  gather_facts: false

  vars:
     engine_url: https://ovirt-engine.example.com/ovirt-engine/api
     engine_user: admin@internal
     engine_password: 123456
     engine_cafile: /etc/pki/ovirt-engine/ca.pem
     
     data_center_name: mydatacenter
     compatibility_version: 4.1
     
     clusters:
      - name: production
        cpu_type: Intel Conroe Family
        profile: production
     
     hosts:
      - name: myhost
        address: 1.2.3.4
        cluster: production
        password: 123456
      - name: myhost1
        address: 5.6.7.8
        cluster: production
        password: 123456
     
     storages:
       mynfsstorage:
         master: true
         state: present
         nfs:
           address: 10.11.12.13
           path: /the_path
       myiscsistorage:
         state: present
         iscsi:
           target: iqn.2014-07.org.ovirt:storage
           port: 3260
           address: 100.101.102.103
           username: username
           password: password
           lun_id: 3600140551fcc8348ea74a99b6760fbb4
       mytemplates:
         domain_function: export
         nfs:
           address: 100.101.102.104
           path: /exports/nfs/exported
       myisostorage:
         domain_function: iso
         nfs:
           address: 100.101.102.105
           path: /exports/nfs/iso
     
     logical_networks:
       - name: mynetwork
         clusters:
           - name: development
             assigned: yes
             required: no
             display: no
             migration: yes
             gluster: no
     
     host_networks:
       - name: myhost1
         check: true
         save: true
         bond:
           name: bond0
           mode: 2
           interfaces:
             - eth2
             - eth3
         networks:
           - name: mynetwork
             boot_protocol: dhcp
     
     users:
      - name: john.doe
        authz_name: internal-authz
        password: 123456
        valid_to: "2018-01-01 00:00:00Z"
      - name: joe.doe
        authz_name: internal-authz
        password: 123456
        valid_to: "2018-01-01 00:00:00Z"
     
     user_groups:
      - name: admins
        authz_name: internal-authz
        users:
         - john.doe
         - joe.doe
     
     permissions:
      - state: present
        user_name: john.doe
        authz_name: internal-authz
        role: UserROle
        object_type: cluster
        object_name: production
     
      - state: present
        group_name: admins
        authz_name: internal-authz
        role: UserVmManager
        object_type: cluster
        object_name: production

  pre_tasks:
    - name: Login to oVirt
      ovirt_auth:
        url: "{{ engine_url }}"
        username: "{{ engine_user }}"
        password: "{{ engine_password }}"
        ca_file: "{{ engine_cafile | default(omit) }}"
        insecure: "{{ engine_insecure | default(true) }}"
      tags:
        - always

  roles:
    - ovirt-infra

  post_tasks:
    - name: Logout from oVirt
      ovirt_auth:
        state: absent
        ovirt_auth: "{{ ovirt_auth }}"
      tags:
        - always
```

[![asciicast](https://asciinema.org/a/112415.png)](https://asciinema.org/a/112415)

License
-------

Apache License 2.0

[ovirt-aaa-jdbc]: https://github.com/oVirt/ovirt-ansible/blob/master/roles/ovirt-aaa-jdbc/README.md
[ovirt-clusters]: https://github.com/oVirt/ovirt-ansible/blob/master/roles/ovirt-clusters/README.md
[ovirt-datacenters]: https://github.com/oVirt/ovirt-ansible/blob/master/roles/ovirt-datacenters/README.md
[ovirt-hosts]: https://github.com/oVirt/ovirt-ansible/blob/master/roles/ovirt-hosts/README.md
[ovirt-networks]: https://github.com/oVirt/ovirt-ansible/blob/master/roles/ovirt-networks/README.md
[ovirt-permissions]: https://github.com/oVirt/ovirt-ansible/blob/master/roles/ovirt-permissions/README.md
[ovirt-storages]: https://github.com/oVirt/ovirt-ansible/blob/master/roles/ovirt-storages/README.md
