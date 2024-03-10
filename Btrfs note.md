


## Snapshot 

Example of making snapshot on home sub volume 

```
sudo btrfs subvolume snapshot /home/ /home/snapshot_$(date --iso-8601=seconds)
```

Example of deleting this snapshot 

```
sudo btrfs subvolume delete snapshot_2023-12-02T23:21:24-06:00
```
 

## fstab

good configuration example with explanations: https://www.jwillikers.com/btrfs-mount-options

Example of my config for a btrfs HDD device. 

```
UUID=63ee14a8-4610-43e5-b42c-d40c02552c18 /mnt/store  btrfs    defaults,autodefrag,nofail,x-systemd.device-timeout=5,compress=zstd:3  0  0
```

`autodefrag` is picked since COW file systems are always bad on fragmentation.

Since This disk is an hdd data disk, I turned on `nofail,x-systemd.device-timeout=5` for it to be ignored if not found during boot. See 3.2 in https://wiki.archlinux.org/title/Fstab for explaining this.

This is a big data storage device, thus i've used the compress option.