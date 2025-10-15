

## V4 single root.

There are some changes about how V4 handls files and stuff. so the way to set NFSv4 share will be slightly different and require extra step on complex folder structure.

Here is an [example from ubuntu](https://help.ubuntu.com/community/NFSv4Howto)


Basically because of single root, there would be the need to do extra `bind` to form things under a single folder. So for a system's setup, need to have `/etc/fstab` entry to map individual folders from any location across the system to a single root folder tree. Then `/etc/exports` to tell setup NFS export of this single root folder tree.

## Disabe V2 v3 

Hit this in the log: 
```
Oct 02 17:44:04 pve1 systemd[1]: Starting nfs-server.service - NFS server and services...
Oct 02 17:44:04 pve1 sh[4326]: nfsdctl: lockd configuration failure
Oct 02 17:44:04 pve1 systemd[1]: Finished nfs-server.service - NFS server and services.
```

edit `/etc/nfs.conf` to turn off V2 and V3

```
[nfsd]
vers3=n
# vers4=y
# vers4.0=y
# vers4.1=y
# vers4.2=y
```

## Sync issue 

https://gist.github.com/fardjad/ea358f9bf844889ecad109b352dd0d5b

Client side `sync` and Server side `sync` setting are not the same settings.

## Opther option to be careful about 

* `wdelay`: The NFS server will delay writing to the disk if it suspects another write request is imminent. This can improve performance as it reduces the number of times the disk must be accessed by separate write commands, thereby reducing write overhead. To disable this, specify the no_wdelay. no_wdelay is only available if the default sync option is also specified.