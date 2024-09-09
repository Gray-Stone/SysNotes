# rsync


## Syncing local drive. 

Copy everything from SRC to DST, force everything in DST be the same as SRC.

```
rsync -v -c --delete -r  src/ dst/

```
-v verbos to see what's going on, specially good with --dry-run

-c only use check-sum, not modified time to skip file

--delete, delete file in DST doesn't exsits in SRC. This is an important one for the usecase of forcing them to be the same.

-r recurisvly do the folders. 

is the src and dst doesn't have the trailing slash, the outcome will be putting src into dst. Adding slash means process the content of the folder, not treating folder as a single entity.


## Special file names.

For file names with lots of special characters, while-space, or wild cards, we need to ensure parsing the contents until at the remote machine. A pair of single quote to prevent argument resolving and the -s will usually do it.

> -s, --protect-args          no space-splitting; wildcard chars only
This is a very good flag for when target destination on remote machine is super strange with spaces and non ascii characters. 

example 

```
╰─➤  rsync --stats --compress --partial --append-verify --progress --human-readable -vr -s --dry-run remote-deviec:'/mnt/files/[abc] special character with white space[s]' ./
```

## Pulling data over internet 


[Online link explaining rsync resume after interrupted](https://unix.stackexchange.com/questions/48298/can-rsync-resume-after-being-interrupted)


```
rsync --stats --compress --partial --append-verify --progress --human-readable -vr --dry-run
```

* give some file-transfer stats
* turn on compression
* Turns on partial transmittoin (when rsync is stopped, normally it will delete a half done file), 
* Append when transmittion resume, and verify it. 
* print progress
* verbose 
* human readable format (with only -h arg means printing help)
* --dry-run (this will list all file needs transfer, but not actually doing it.) Remove this when actual transmission is about to start