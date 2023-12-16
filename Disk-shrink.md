# Shrink the partition on disk

---
## NTFS

## GUI 

The easy way is to do it through gparted. 

Internally it will use `ntfsresize` which is the actual command for shrinking NTFS 

## Failure

After shrinking failed, when mounting it directly without argument 

```
╰─➤  sudo mount /dev/nvme1n1p4 /mnt/tmp-mount 
Mount is denied because the NTFS volume is already exclusively opened.
The volume may be already mounted, or another software may use it which
could be identified for example by the help of the 'fuser' command.
```

However, this is a fake message, the real one is: 

```
╰─$ sudo mount -t ntfs3 /dev/nvme1n1p4 /mnt/tmp-mount -o remove_hiberfile     130 ↵
mount: /mnt/tmp-mount: 文件系统类型错误、选项错误、/dev/nvme1n1p4 上有坏超级块、缺少代码页或帮助程序或其他错误.
       dmesg(1) may have more information after failed mount system call.

or 

mount: /mnt/Data: wrong fs type, bad option, bad superblock on /dev/sdb1, missing codepage or helper program, or other error.
```

Looks like the partition is marked dirty, which require it to be booted into windows for a disk check. 

### Fixing it in linux

If the drity bit is set for trivel issue, there might be a way out, but not very likely in this case. 

Usually `fsck` is used to fix disks. 

https://askubuntu.com/questions/821745/what-is-the-linux-equivalent-for-the-windows-chkdsk-command
https://help.ubuntu.com/community/FilesystemTroubleshooting

However, ntfs only have `ntfsfix` command. 

```
╰─$ sudo ntfsfix /dev/nvme1n1p4                                                 1 ↵
Mounting volume... Windows is hibernated, refused to mount.
FAILED
Attempting to correct errors... 
Processing $MFT and $MFTMirr...
Reading $MFT... OK
Reading $MFTMirr... OK
Comparing $MFTMirr to $MFT... OK
Processing of $MFT and $MFTMirr completed successfully.
Setting required flags on partition... OK
Going to empty the journal ($LogFile)... OK
Windows is hibernated, refused to mount.
Remount failed: Operation not permitted
```

Basically, the only option for directly doing it in linux is to force set the dirty bit, (which is likely to be bad)

### Mounting it in a windows vm

Have virt-manager attach this device directly to the vm. Then go into the windows vm, under property for the disk, do a scan and repair. 

And this works. 
