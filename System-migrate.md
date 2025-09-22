Migrating system:

## destination disk

First make sure destination harddrive is partitioned the desire way. 

For btrfs, makes and mount the subvolumes. 

```
/mnt/new-temp
├── home
├── snapshots
└── var
└── log
```

```
╰─➤  sudo btrfs subvolume list /mnt/new-temp
ID 257 gen 20 top level 5 path @
ID 258 gen 18 top level 257 path home
ID 260 gen 19 top level 257 path var/log
ID 261 gen 20 top level 257 path snapshots
```


Then rsync entier machine over:

```
rsync -aHAXS --stats --progress --partial --append-verify -x /mnt/ /. /mnt/new-temp/
```

note: this is extreamly verbous.