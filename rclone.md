# rclone 

This is yet another sync/backup/copy utility.

The keep selling point for using it is its large number of supported backends.

It would be used as is for backup into cloud, or combine with other tools to send backup to cloud.



## Config

rclone needs to be configured before using. For each destination, a remote needs to be added to config 


```
rclone config
```

Will start this process. 

You can think of each remote as a remote host name when using ssh based tool (like scp)

for example when copying files using rclone 

```
rclone copy file_folders onedrive-remote1:/file_folders
```

will copy files to onedrive-remote1, which is a user picked name during configuration.

User will need to handle authentication and etc during config, after finished, a backend is added and can be used in other commands.

## Copy 

https://rclone.org/commands/rclone_copy/

This is used when simply want to dump files onto cloud, and not deleting anything on cloud side. 

## sync 

https://rclone.org/commands/rclone_sync/

This is to sync destination to be the same and source. extra files in destination will be deleted.

for example

```
rclone copy --progress 媒体 onedrive-back2:/媒体 --dry-run
```
