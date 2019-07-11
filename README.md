## Description

Ansible playbooks to migrate a single root partition scheme or volume to DISA STIG/CIS approved LVM layout.

## Requirements

A second disk attached to your machine/instance.
Examples:
* Physical disk
* Virtual hard disk
* AWS EBS volume

## Migration Process

Step 1 - Create LVM layout:
```
ansible-playbook -i inventory --extra-vars='new_disk=/dev/sdb' lvm2.yml
```

Step 2 - Rsync current disk to new disk:
```
ansible-playbook -i inventory sync.yml
```

Step 3 - Update fstab:
```
sudo ansible-playbook -i inventory --extra-vars='new_disk=/dev/sdb' fstab.yml
```

Step 4 - Update grub config:
```
sudo ansible-playbook -i inventory --extra-vars='new_disk=/dev/sdb' grub.yml
```
## Variable Override Examples

* Check vars/main.yml for additional override options

CentOS/RHEL 6 - ext4 vs default xfs for root:
```
ansible-playbook -i inventory --extra-vars='new_disk=/dev/sdb' --extra-vars='root_fs=ext4' lvm2.yml
```

RHEL 8 - xfs vs default ext4 for boot:
```
ansible-playbook -i inventory --extra-vars='new_disk=/dev/sdb' --extra-vars='boot_fs=xfs' lvm2.yml
```

No swap created on LVM:
```
sudo ansible-playbook -i inventory --extra-vars='new_disk=/dev/sdb' --extra-vars='create_swap=false' grub.yml
```

CIS disk layout to add /var/tmp:
```
ansible-playbook -i inventory --extra-vars='new_disk=/dev/sdb' --extra-vars='cis_layout=true' lvm2.yml
```

## License and Author

* Author:: Nick Adams
