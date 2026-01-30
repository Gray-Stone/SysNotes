
## To see holds on snapshots 

`zfs get -H -t snapshot -o name,value userrefs`

This will list all the snapshots, and print their name and `userrefs` count.  `userrefs` count would show the actual number of holds on each snapshot.

`zfs holds -r <dataset@snapshot>` to see the holds on the snapshot.