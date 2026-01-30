
## Torrent Setup 

Generally torrenting is a 2 part setup. A inprogress dataset and A finished dataset.

In progress dataset would handle more fragmented data, etc. The finishd dataset could have larger record size for non-changing data that's mostly read only. 
Also the "move" from in progress to finish also re-writes all data, effectively reducing the fragmentation a lot, and allowed the new dataset settings to take effect.


### In Progress

The actual datasize itself might not be as relevent, but some other setting could help more.


There's the opinion to disable sync to help reduce small fagrment disk writes.

https://github.com/openzfs/zfs/discussions/15287#discussioncomment-7025344



```
sudo zfs create -o xattr=sa -o dnodesize=auto -o atime=off -o compression=zstd -o recordsize=16K -o sync=disabled RedPool/qb-temp
```

* atime off, don't care about read time here.
* recordsize, a smaller recordsize would match better to software, 
* sync=disabled This bascially means we don't value the last bits of data from a partially downloaded torrent.

Other possible options 
* primarycache=metadata This is only when you are sure there won't be much uploading when downloading.













sudo zfs create -o xattr=sa -o dnodesize=auto -o relatime=on -o compression=zstd -o recordsize=256k RedPool/media

sudo zfs create -o atime=off -o compression=zstd-6 -o recordsize=1M RedPool/media/Ext

sudo zfs create -o relatime=on -o recordsize=256K RedPool/media/Ext/music

sudo zfs create -o xattr=sa -o dnodesize=auto -o relatime=on -o compression=zstd-6 RedPool/media/Int