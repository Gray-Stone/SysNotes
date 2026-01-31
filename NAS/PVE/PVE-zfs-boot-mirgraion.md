

## Starting point

Current system is a single large-ish ssd as proxmox boot and system disk.

We want to transform into a zfs mirror with 2 ssds, and sadly, the new disk is smaller. But the overall system disk usage has not exceed the second device's size.

## Preapre the mirror.

First make the data exists on the second drive, this is done while server still on, to reduce later offline switch over time.

### 
Setup the new disk. 

Basically make 2 partitions, first a EFI partition, from start to 512M, then a second empty one filling the rest of the space.

Then Create a new zfs pool on second partition, this partition needs a different name, so no collision. Will change it back at the very end.

Somehow it seems ok to have different property and stuff, including compression ratio.

```
zpool create -f -o ashift=12 -o autotrim=on \
  -O atime=off -O xattr=sa -O acltype=posixacl \
  -O compression=off -O mountpoint=none \
  rpoolNEW ${NEW}-part2
```

### First patch zfs send.

Make a snapshot on main pool, then transfer this snapshot over. This will help cutdown the transfer time after the power-cut switch-over.

Make the snapshot:
```
zfs snapshot -r rpool@move-pre
```

Send that snapshot over.
```
zfs send -R rpool@move-pre  | sudo  zfs receive -Fdu -o compression=lz4 rpoolNEW
```

## Shutdown and switch over 

Shutdown the machine now.

Boot into an alternative OS, usually a live usb boot. 

AI recommanded proxmox ISO, but I can't get into it's terminal mode, so i'm using arch liveboot usb.

This repo will help to get openzfs loaded on a live arch iso. Even have helper command for already booted arch iso.

https://github.com/stevleibelt/arch-linux-live-cd-iso-with-zfs/?tab=readme-ov-file


### Mount both and do a final sync.


```
zpool import -f rpool -R /mnt/src
zpool import -f -d /dev/disk/by-id rpoolNEW -R /mnt/target
```

Under `src` and `target` should be the full disk, with one difference being `src` have `rpool` inside, and `target` have `rpoolNEW` inside.

now take snapshot and send increment 
```
zfs snapshot -r rpool@move-final-iso
zfs send -RI rpool@move-pre rpool@move-final-iso | zfs receive -Fdu -o compression=lz4 rpoolNEW
```

### Rename

export the old pool, so it's name won't interfere with renaming.
```
zpool export rpool
```

`zpool status` should not have a zpool listed now.


Export rpoolNEW so we can re-import it with rename.
```
zpool export rpoolNEW
```

Now reimport `rpoolNEW` but as `rpool` name 
```
zpool import -f -d /dev/disk/by-id rpoolNEW rpool -R /mnt/target
```

### Migrate the system

```
zfs mount -a
```
This make sure all datasets are mounted, necessary for chroot stuff later.

 
```
zfs list -o name,mountpoint
```

This to double check if mounted correctly. Should be able to see the proxmox's zfs datasets


```
for d in dev proc sys run; do mount --rbind /$d /mnt/target/$d; done
chroot /mnt/target /bin/bash
```
This mounts all the runtime folders for proxmox under `/mnt/target`, then chroot over.

Then `ls` command should show the old machine's layout.


```
zpool set cachefile=/etc/zfs/zpool.cache rpool
```
This is needed for initramfs / systemd to import the rpool's topology correctly 

### Resetting the EFI boot thing.

AI's initial command stack did a whole thing with `proxmox-boot-tool`, This tool initial doesn't exists because my `PATH` variable didn't include the `/usr/sbin`. So the following section until updating initramfs have 2 options.

The two option creates essentially the same thing, with fist automated tool getting fancier result with multiple entries.


#### A: use proxmox-boot-tool

```
proxmox-boot-tool format /dev/disk/by-id/<500G>-part1 --force
proxmox-boot-tool init   /dev/disk/by-id/<500G>-part1
proxmox-boot-tool refresh
```


#### B: manually do it.

which doesn't exists on the system (even in the /mnt/target's path.) So doing a alternative with systemd-boot

Set the env-var for the new disk's efi partition (part1 here since I created the new disk with first partition as EFI)
```
ESP=/dev/disk/by-id/nvme-xxxxxxxxxxxxxxxx-part1
```

mount it `mount "$ESP" /boot/efi`.


Install systemd-boot into the ESP (and create NVRAM entry)
```
bootctl --path=/boot/efi install
```


Pick out latest kernal and initramfs

AI used a combo shell command, I simply do the `ls -l` and manually read it. Which the thing matches.
```
K=$(basename "$(ls -1 /boot/vmlinuz-*-pve | sort -V | tail -n1)")
I=$(basename "$(ls -1 /boot/initrd.img-*-pve | sort -V | tail -n1)")
```

copy them over
```
mkdir -p /boot/efi/EFI/proxmox
cp -av "/boot/$K" /boot/efi/EFI/proxmox/
cp -av "/boot/$I" /boot/efi/EFI/proxmox/
```

then write loader config 
```
mkdir -p /boot/efi/loader/entries
cat >/boot/efi/loader/loader.conf <<'EOF'
default proxmox.conf
timeout 5
editor no
EOF

cat >/boot/efi/loader/entries/proxmox.conf <<EOF
title   Proxmox VE (ZFS)
linux   \\EFI\\proxmox\\$K
initrd  \\EFI\\proxmox\\$I
options root=ZFS=rpool/ROOT/pve-1 boot=zfs rw
EOF
```

This could be done with editor like vi, the command is to replace the content with new string. Note: the second section there are text variable replacement.

### now rebuild initramfs
Likely would run into command not found because `/usr/sbin` is not in the path, make sure to reset `PATH`  `export PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin` 

```
update-initramfs -u -k all
```
Terminal should scroll with some info related to the kernel we setup in previous steps.

### last setting on zfs 

Since my previous boot disk don't have any optional setting, I'm adding this extra compression setting as default on the new pool

```
zfs set -r compression=lz4 rpool
```


### Checks 

This would list the entries we just set.
```
proxmox-boot-tool status
```

`cat /etc/hostid` this returned empty for me, so we need to copy it from the old one.

### Fixing hostid for new pool

we need to import the old pool again, since the new pool is already have it's name changed, they have name conflict now. need to import old pool by specific device and named it to new name 

the -N will delay the mount later.

also for some reason my initial PVE install the boot is on partition 3. (partiion 1 is 1000K part.)
```
zpool import -f -N \
  -d /dev/disk/by-id/nvme-INTEL_xxxxxxxxxxxxx_part3 \
  -o readonly=on -o cachefile=none \
  -R /mnt/src \
  rpool rpoolOLD


zfs mount -a
```

double check the old hostid `hexdump -C /mnt/src/etc/hostid` this should show a 4 byte binary.

then copy it `cp /mnt/src/etc/hostid /mnt/target/etc/hostid` 

now export the old pool `zpool export rpoolOLD`

chroot into new system again `chroot /mnt/target /bin/bash` , double check again with `hexdump -C /etc/hostid` 


regen initramfs 

```
zpool set cachefile=/etc/zfs/zpool.cache rpool
update-initramfs -u -k all
```


### Clean up then reboot

Exit the chroot, then umount things 

```
for d in run sys proc dev; do mount --make-rslave /mnt/target/$d; umount -R /mnt/target/$d; done
```

make sure to export the pool `zpool export rpool` . If zpool said pool is buy, double check `/boot/efi` inside is unmounted, also do `zfs unmount -a` . They it should be good.


## Boot into new disk

Now we call `reboot` in live iso and boot into the new disk.

It could be confusing on some system as the two boots are exactly the same name. At this point, go into bios.

After booted, we should have our entire proxmox working again, using `zpool status` should show the loaded rpool is from new drive.

## Now make the old drive into mirrors.

First wipe the old drive to the same layout as our new drive (which is smaller).

```
sudo fdisk /dev/disk/by-id/<old_disk_name>

Welcome to fdisk (util-linux 2.41).
Changes will remain in memory only, until you decide to write them.
Be careful before using the write command.


Command (m for help): g
Created a new GPT disklabel (GUID: 9A291030-793E-4247-A7CA-ABB87E01AE41).

Command (m for help): n
Partition number (1-128, default 1): 1
First sector (2048-2344225934, default 2048):
Last sector, +/-sectors or +/-size{K,M,G,T,P} (2048-2344225934, default 2344224767): +512M

Created a new partition 1 of type 'Linux filesystem' and of size 512 MiB.
Partition #1 contains a vfat signature.

Do you want to remove the signature? [Y]es/[N]o: Y

The signature will be removed by a write command.

Command (m for help): t
Selected partition 1
Partition type or alias (type L to list all): 1
Changed type of partition 'Linux filesystem' to 'EFI System'.

Command (m for help): n
Partition number (2-128, default 2): 2
First sector (1050624-2344225934, default 1050624):
Last sector, +/-sectors or +/-size{K,M,G,T,P} (1050624-2344225934, default 2344224767):

Created a new partition 2 of type 'Linux filesystem' and of size 1.1 TiB.

Command (m for help): w
The partition table has been altered.
Calling ioctl() to re-read partition table.
Syncing disks.

```

Then format the EFI parition

```
mkfs.vfat -F32 /dev/disk/by-id/<old_disk_name>
```

Then construct a zfs mirror from both drive.
```
sudo zpool attach rpool \
  /dev/disk/by-id/<new_disk_name>-part2 \
  /dev/disk/by-id/<old_disk_name>-part2
```
A catch here is since proxmox is fully online, I am doing this over ssh in user shell, without sudo, zpool throw a misleading message about disk having vfat format already. But using sudo fixed it.

### Also mirror the EFI partition 

Same command as we have done before. nothing special.

```
proxmox-boot-tool format /dev/disk/by-id/<old_disk_name> --force 
proxmox-boot-tool init   /dev/disk/by-id/<old_disk_name>
proxmox-boot-tool refresh
```