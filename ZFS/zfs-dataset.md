# Dataset

Dataset is like a another "device" inside pool, which have attribute could be controlled, individually snapshotted. Also nest-able.

Each large section under pool should be made as dataset. 

## Common Properties

### Options might need setting.

from [This Post](https://serverfault.com/questions/1143267/what-zfs-pool-vdev-and-dataset-properties-should-most-people-use)

* `xattr=sa` This is a linux specific option. not default on for compatibility [ZFS-commit-note](https://github.com/openzfs/zfs/commit/82a37189aac955c81a59a5ecc3400475adb56355)
* `dnodesize=auto` Paired with `xattr=sa`
* `relatime=on` save operation on read only action by less accurate atime, already common on other disk format. (or turn atime off if don’t care).
* `compression=xxx` Compression have almost no draw back, only benefit. So turn it on.
* `recordsize=xxx` The default value is a good middle group. but on very specific type of dataset like Large media only case, tuning this could have some benefits.

### Options shouldn't use

Dedupe should not be turned on. It is very heavy on CPU and RAM. and it's a block level dedupe, normally files won't have too much duplications unless on things like node modules. A cheap way to have some dedupe manually is simply use links instead of copying files around.

## Compression

How much compression could have really depends on the files. Things like video files are mostly non-compress-able. Things like text are highly compress-able.

`zstd` and `lz4` are the two common options that are good formance. They are both fast.

zstd specially have levels could be selected. It's designed to always de-compress fast, but compresstion level could give you trade off between speed and higher compression.

[A Post from TrueNas form](https://www.truenas.com/community/threads/zstd-speed-ratio-benchmarks.89429/) comparing different zstd compression level, also including lz4 in comparison. 

Data from this showed `lz4` is faster, compress less in general. Also for compressible data, at `zstd=6` is a plateau in terms of compression ratio until much higher level.

## Mountpoint

Concept of mountpoint: [OpenZFS # Concept # Mount_Points](https://openzfs.github.io/openzfs-docs/man/master/7/zfsconcepts.7.html#Mount_Points)

The `mountpoint` of a dataset is automatically set. If using `-o mountpoint=/some/path` , the file system will be mounted to the specificed spot. 

`mountpoint` property is inherited, so nested dataset is by default, auto mounted into the parent's path. Example from openzfs document: 
> so if pool/home has a mount point of `/export/stuff`, then pool/home/user automatically inherits a mount point of `/export/stuff/user`.

Since there could be lots of datasets in a system, zfs auto manage mounting without needing `/etc/fstab` file. mountpoint is recorded as dataset property and automatcially mount by zfs at boot or when `zfs mount -a` is called.

If prefer manually manage it, could set the mount option to legacy, then zfs won't auto mount it. But dataset can only be mounted after zfs pool is imported. On machine with systemd, `x-systemd.requires=zfs-import.target` can help ensure zfs-import is done before systemd mount dataset.

Note: the zfs mount point is written as a property, when a pool is moved to a new machine, with different folder layout, this mountpoint might collide with other folders. And depending on systems and parameters, it might overlay existing directory or fail to auto mount. Might need `altroot` to help avoid this.


## Record Size

[OpenZFS Tuning](https://openzfs.github.io/openzfs-docs/Performance%20and%20Tuning/Workload%20Tuning.html#dataset-recordsize)

Dataset is the basic unit of data used for internal copy-on-write on files. Partial record write require data be read-modify-write, as it changes the entier chunk. Could even think of the ZFS recordsize something similiar to SSD's 4k align, at a software level, in larger size.

If software write data in fixed size chunk, it'll be best to match the record size. Recordsize could be changed later, but only effects newer data. Need to re-create files if need to update recordsize for existing files (or send/recv)

Default dataset use recordsize of 128KB by default. This is generally good for mixed usage.Newer system should default support up to 16M record size (require large_blocks pool feature)



### What size to use

What recordsize to use really depends on useage, there are lots of opinion floating around. 

One simply option is to only change this on *NEED TO CHANGE* baiese after hitting a problem.

* https://serverfault.com/questions/1117662/disadvantages-of-using-zfs-recordsize-16k-instead-of-128k/1120640#1120640

Generally it's common to use large dataset size for Large file dataset, like Media section.

[OpenZFS guide](https://openzfs.github.io/openzfs-docs/Performance%20and%20Tuning/Workload%20Tuning.html#bit-torrent) also suggest Bit Torrent to use 16KB recordsize, as it does lots of 16KB write. It's best to move the finished download data to a new location so the data gets a rewrite and becomes sequential and compact after words. New location could also be a dataset with different settings.


