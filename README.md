lvm-management
=========

Manage lvm volume groups and volumes with their respective fstypes and mountpoints

Requirements
------------

None (yet...)

Role Variables
--------------
```yaml
lvm_management_default_fstype: ext4
```
Default filesystem to use to format logical volumes unless otherwise stated 

```yaml
lvm_management_default_fstab_dump: 0
```
Default value for dump field (5th) in fstab. See man fstab

```yaml
lvm_management_default_fstab_passno: 0
```
Default value for passno field (6th) in fstab. See man fstab

```yaml
lvm_management_default_vg: vg01
```
Default volume group for logical volume creation unless otherwise stated

```yaml
lvm_management_vgs: []
# Example vg definition
# - name: vg01
#   pvs: /dev/sdb,/dev/sdc
```
List of logical volumes groups to create. Empty by default. Each VG in the list must define:
* `name` (required): the name of the Volume Group
* `pvs` (required): coma separated list of physical volumes to use. PVS will be automatically initialized for use with lvm (pvcreate) when needed.

```yaml
# List of logical volumes to be created
# List higest mountpoints firts (i.e. /mount before /mnt/sub/mount)
lvm_management_lvs: []
# Example lv definition
# - name: lv_01
#   size: 1G
#   mountpoint: /mnt/volume1
#   vg: vg_02
#   fstype: xfs
#   fstab_dump: 1
#   fstab_passno: 2
#   fstab_opts: noauto
```
List of logical volumes to create. Each lv in the list defines:
* `name` (required): Name of the logical volume
* `size` (required): The size of the logical volume (see -L option of lvcreate). _Resizing volumes has not been tested yet._
* `mountpoint` (required): The path to the mount point. It will be created if it does not exist. **If you have submounts, make sure to list the highest mountpoints first in the list** (i.e. /rootmount before /rootmount/submount).
* `fstype` (optional): Filesystem to use. Volume will be formated when needed. defaults to `lvm_management_default_fstype`. _Resizing and/or changing fstype has not been tested yet_.
* fstab_dump (optionnal): Value of dump field (5th) in fstab. See `man fstab`. Will default to `lvm_management_default_fstab_dump`
* fstab_passno (optionnal): Value of passno field (6th) in fstab. See `man fstab`. Will default to `lvm_management_default_fstab_passno`
* fstab_opts (optionnal): Value of mount option field (4th) in fstab. See `man fstab`. Will default to "defaults".

Dependencies
------------

* Must be ran on a physical machine or a VM
* OS with lvm support

Example Playbook
----------------

```yaml
- hosts: all
  remote_user: deployuser
  become: yes

  roles:
    - lvm-management

  vars:

    lvm_management_default_fstype: xfs
    lvm_management_default_fstab_dump: 1
    lvm_management_default_fstab_passno: 2
    lvm_management_default_vg: main_vg

    lvm_management_vgs:
      - name: "{{ lvm_management_default_vg }}"
        pvs: /dev/sdb
      - name: secondary_vg
        pvs: /dev/sdc,/dev/sdd
    lvm_management_lvs:
      - name: application_root
        size: 20G
        mountpoint: "/opt/application"
      - name: application_log
        size: 5G
        mountpoint: "/opt/application/log"
      - name: application_data
        size: 50G
        mountpoint: "/opt/application/data"
      - name: backups
        size: 200G
        fstype: ext4
        mountpoint: "/backups"
        vg: secondary_vg
        fstab_dump: 0
        fstab_passno: 1
        fstab_opts: defaults,umask=0
```

License
-------

MIT

Author Information
------------------

olivier.clavel@thoteam.com
