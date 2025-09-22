


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


## Snapshot layout 

Nested snapshot is almost like normal folders where it auto mount. However it saves manually moutning each subvolume to specific directory.
https://bbs.archlinux.org/viewtopic.php?id=249321

Despite the nesting, when taking snapshot, each subvolume are still threated separately. Which mean this could also be a trick to isolate folder within nesting to prevent snapshotting.

https://wiki.archlinux.org/index.php/Dm-crypt/Encrypting_an_entire_system#Create_nested_subvolumes

### 

When subvolume are created, they are directly created as folders.
`sudo mount /dev/sde3 /mnt/new-temp`
`sudo btrfs subvolume create /mnt/new-temp/@` 
`sudo btrfs subvolume create /mnt/new-temp/@home` 
restuls in 

```
╰─➤  sudo tree /mnt/new-temp
/mnt/new-temp
├── @
│   └── home
│       └── a.f
└── @home
```

If mounted like `sudo mount -o subvol=@ /dev/sde3 new-temp` 
Then the folder structure will be, where the flat layout @home subvolume is not shown.

```
/mnt/new-temp
└── home
└── a.f
```
