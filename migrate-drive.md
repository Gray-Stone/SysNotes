Used cloinzilla to move my system from 1 drive to another

clonezilla will replicate even the disk uuid to be exactly the same 

However, it does not clone the FAT32 boot partition correctly. 

Old disk layout 

fat32 -> boot,efi 
btrfs -> root, /

This structure is perfectly migrated to the new drive. 

However, when booting with just new drive, bios did not find any bootable drive.

When putting both disk into the system, it booted. This is just an extreamly lucky case, It shouldn't have worked because the system have 2 sets of partitions with same uuid. it's just pure luck 

After booted in, looking at gparted, seems like the system booted using old disk's fat32 partition. But booted into new disk's btrfs partition. 

After looking closely, gparted doesn't seems to be able to read the fat32 partitiion of new disk. Seems like clonezilla didn't do a good job on cloning that. 

I used dd command to clone the boot partition myself. Just one dd command, didn't regenerate any gurb or efi stuff. 

Then it worked! new drive can boot by itself.