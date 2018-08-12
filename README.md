# ovirt-ansible-hosted-engine-setup

Ansible role for deploying oVirt Hosted-Engine

## PLEASE NOTE

This package is still in development. The oVirt Project makes no guarantees as to its suitability or usefulness.

This pre-release should not to be used in production, and it is not feature complete.

# Requirements

 * Ansible version 2.6 or higher
 * Python SDK version 4.2 or higher

# Dependencies

No.

# Prerequisites

* Add the official oVirt repository:
```sh
$ sudo yum install http://resources.ovirt.org/pub/yum-repo/ovirt-release42.rpm
```
* A fully qualified domain name prepared for your Engine and the host. Forward and reverse lookup records must both be set in the DNS.
* `/var/tmp` has at least 5 GB of free space.
* Unless you are using Gluster, you must have prepared storage for your Hosted-Engine environment (choose one):
    * [Prepare NFS Storage](https://ovirt.org/documentation/admin-guide/chap-Storage/#preparing-nfs-storage)
    * [Prepare ISCSI Storage](https://ovirt.org/documentation/admin-guide/chap-Storage/#preparing-iscsi-storage)


# Example Playbook
This is a simple example for deploying Hosted-Engine with NFS storage domain.

All the playbooks can be found inside the `examples/` folder.

## hosted_engine_deploy.yml

```yml
---
- name: Hosted-Engine-Setup_Part_01
  hosts: localhost
  connection: local
  vars_files:
    - passwords.yml
  vars:
    he_install_packages: true
    he_pre_checks: true
    he_initial_clean: true
    he_bootstrap_local_vm: true
  roles:
    - role: oVirt.hosted-engine-setup

- name: Hosted-Engine-Setup_Part_02
  hosts: engine
  vars_files:
    - passwords.yml
  vars:
    he_bootstrap_local_vm_engine: true
  roles:
    - role: oVirt.hosted-engine-setup

- name: Hosted-Engine-Setup_Part_03
  hosts: localhost
  connection: local
  vars_files:
    - passwords.yml
  vars:
    he_bootstrap_local_vm_add_host: true
    he_create_storage_domain: true
    he_create_target_vm: true
  roles:
    - role: oVirt.hosted-engine-setup

- name: Hosted-Engine-Setup_Part_04
  hosts: engine
  vars_files:
    - passwords.yml
  vars:
    he_engine_vm_configuration: true
  roles:
    - role: oVirt.hosted-engine-setup

- name: Hosted-Engine-Setup_Part_05
  hosts: localhost
  connection: local
  vars_files:
    - passwords.yml
  vars:
    he_final_tasks: true
    he_final_clean: true
  roles:
    - role: oVirt.hosted-engine-setup
```

### __Note__:
Unlike standard roles that are called once in a playbook, we call this role 5 times. The reason for that is due to the fact that Ansible allows executing a role only for one host. Thus, because we need to execute tasks on both `host` and `engine` hosts we had to call the role more than one time.

## password.yml

```yml
---
# As an example this file is keep in plaintext, if you want to
# encrypt this file, please execute following command:
#
# $ ansible-vault encrypt passwords.yml
#
# It will ask you for a password, which you must then pass to
# ansible interactively when executing the playbook.
#
# $ ansible-playbook myplaybook.yml --ask-vault-pass
#
he_appliance_password: 123456
he_admin_password: 123456
```

## nfs_deployment.json ( extra vars for NFS deployment ):

```json
{
    "he_bridge_if": "eth0",
    "he_fqdn": "he-engine.example.com",
    "he_vm_mac_addr": "00:a5:3f:66:ba:12",
    "he_domain_type": "nfs",
    "he_storage_domain_addr": "192.168.100.50",
    "he_storage_domain_path": "/var/nfs_folder"
}
```

# Usage
1. Check all the Prerequisites and requirements are met.
2. Encrypt passwords.yml
```sh
$ ansible-vault encrypt passwords.yml
```

3. Execute the playbook (for NFS deployment)

```sh
$ ansible-playbook hosted_engine_deploy.yml --extra-vars='@nfs_deployment.json' --ask-vault-pass
```

# License

Apache License 2.0