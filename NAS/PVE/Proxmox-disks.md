
## ZFS

A zfs pool can be used to store VM disk. The only format available is raw, which seems to create a zvol on actual zfs disk. This allows the VM to be snapshotted, which is very useful on live-backup, which will be snapshot 

```
root@pmox-node-1:# zfs list
NAME                         USED  AVAIL  REFER  MOUNTPOINT

rpool                        404G  53.1G   104K  /rpool
rpool/ROOT                  31.0G  53.1G    96K  /rpool/ROOT
rpool/ROOT/pve-1            31.0G  53.1G  31.0G  /
rpool/data                   266G  53.1G    96K  /rpool/data
rpool/data/vm-100-disk-0     332K  53.1G   148K  -
rpool/data/vm-100-disk-1    40.4G  53.1G  20.7G  -
rpool/data/vm-100-disk-2     192K  53.1G    64K  -
rpool/data/vm-101-disk-0     100K  53.1G   100K  -
rpool/data/vm-101-disk-1    42.0G  53.1G  42.0G  -
rpool/data/vm-101-disk-2      64K  53.1G    64K  -
rpool/data/vm-102-disk-0     416K  53.1G   156K  -
rpool/data/vm-102-disk-1    59.2G  53.1G  51.3G  -
bluepool                     247G   203G  51.5G  /bluepool
bluepool/subvol-109-disk-0   516M  3.50G   516M  /bluepool/subvol-109-disk-0
bluepool/vm-105-disk-0         3M   203G   152K  -
bluepool/vm-105-disk-1      65.0G   217G  50.7G  -
bluepool/vm-105-disk-2         6M   203G    64K  -
bluepool/vm-107-disk-0         3M   203G   148K  -
bluepool/vm-107-disk-1      65.0G   214G  53.6G  -
```

## Thin Provisioning

For the above zfs status, the VM are allowed 64G of disk space max. 

`rpool` have thin provisioning turned on, so zvol did not used up all 64G of storage. 

However `bluepool` forgot to turn this on, thus resulting in all disk on blue pool used up the entire 64G per VM. Thin provisioning can be turned on after the fact, but won't effect existing disks. Need to go into the specific VM page -> Hardware, click on the Hard Disk, under disk action -> move storage. and select the destination the same pool as current. After move is done, the disk usage will shrink back down.

## Cluster Directory: 

Often a pool need to have a directory added inside of it to be able to store backup and other types onto it. (default after install, there'll be the `local` directory already made). When creating this, make sure to double check `Nodes:` option. Default is `All` which will cause the same path of folder be created on all nodes. 

This is a high availability oriented design, so when migrating a drive with local storage, the destination node will have all the same directories.

**It will also happily create all the parent dir if you forgot to do so. Which mean if you want it to be a sub folder under a ZFS pool, make sure spelling is correct.**