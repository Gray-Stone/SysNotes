# ZFS NFS 

[In the manual](https://openzfs.github.io/openzfs-docs/man/master/7/zfsprops.7.html#sharenfs), and many other places, there is a mention of if wanting to export a zfs dataset over nfs, one should do so via the zfs dataset's feature `sharenfs`


## Old fashion way

### Create two dataset 
```
sudo zfs create -o xattr=sa -o dnodesize=auto -o relatime=on -o compression=lz4 Pool/some_files

sudo zfs create -o xattr=sa -o dnodesize=auto -o relatime=on -o compression=lz4 Pool/some_files/nested1
```

### fstab bind

Create fstab to bind them onto the NFSv4 single root (because I have other directories from other places in future)

The destination folder `/nfs-rt/some_files/` are pre created.

```
/Pool/some_files /nfs-rt/some_files/  none bind,x-systemd.requires=zfs-mount.service 0 0
```

note: I only binded the `some_files` folder, I'm letting the nesting folder automatcially falls into place. 

### exports


```
/nfs-rt          192.168.10.0/24(rw,fsid=root,insecure,no_root_squash,no_subtree_check,sync)

/nfs-rt/some_files    192.168.10.0/24(rw,insecure,no_root_squash,no_subtree_check,sync)

```

There needs to have a top level root folder entry.

Note I actually only exporeted the `some_files` folder. Testing this by mounting `server:/some_files` works just file, the nested dataset is detected and able to read write from it.


## New ZFS way.

Not tested yet, no idea. 

[Here is an example](https://blog.programster.org/sharing-zfs-datasets-via-nfs)