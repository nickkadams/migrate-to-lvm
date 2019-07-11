## Description

Ansible playbooks to migrate a single root partition scheme or volume to DISA STIG/CIS approved LVM layout.

## Requirements

A known good backup if running against a production system.

A second disk attached to your machine/instance.
Examples:
* Physical disk
* Virtual hard disk
* AWS EBS volume

## Tested Operating Systems

* CentOS 6.10
* CentOS 7
* RHEL 6.10
* RHEL 7.6
* Debian 9.9
* Ubuntu 16.04
* Ubuntu 18.04

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

Step 5 - Power off instance and detach original volume.

Step 6 - Attach secondary volume as primary and power on.

## Variable Override Examples

* Check vars/main.yml for additional override options

AWS EC2 with NVMe:
```
ansible-playbook -i inventory --extra-vars='new_disk=/dev/nvme1n1' --extra-vars='boot_part=p1' --extra-vars='root_part=p2' lvm2.yml
```

CentOS/RHEL 6 - ext4 vs default xfs for root:
```
ansible-playbook -i inventory --extra-vars='new_disk=/dev/sdb' --extra-vars='root_fs=ext4' lvm2.yml
```

RHEL 8 - xfs vs default ext4 for boot:
```
ansible-playbook -i inventory --extra-vars='new_disk=/dev/sdb' --extra-vars='boot_fs=xfs' lvm2.yml
```

Skip swap created on LVM:
```
sudo ansible-playbook -i inventory --extra-vars='new_disk=/dev/sdb' --extra-vars='create_swap=false' grub.yml
```

CIS disk layout to add /var/tmp:
```
ansible-playbook -i inventory --extra-vars='new_disk=/dev/sdb' --extra-vars='cis_layout=true' lvm2.yml
```

## Gotchas

* On AWS EC2, when reattaching the secondary EBS volume as the primary volume, use /dev/sda1.
* On VMware vCenter, when removing both disks DO NOT select "Delete files from datastore". Then Add New Device > Existing Hard Disk. Choose YOUR_VM_1.vmdk as SCSI 0:0.
* CentOS/RHEL 6 only updates grub/initrd for default kernel.
* RHEL 8 gather_facts is noticeably slower, in my testing, for ansible chroot operations (Steps 3 and 4).

## License and Author

* Author:: Nick Adams
