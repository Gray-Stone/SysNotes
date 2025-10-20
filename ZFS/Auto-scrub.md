With the proxmox installed, there actually is a included tempalted timer unit for srubbing. 

https://manpages.debian.org/testing/zfsutils-linux/zpool-scrub.8.en.html

```
% systemctl list-unit-files '*scrub*@*.timer'
UNIT FILE                STATE    PRESET
zfs-scrub-monthly@.timer disabled enabled
zfs-scrub-weekly@.timer  disabled enabled
```

the content also looked simple 

```
% cat /lib/systemd/system/zfs-scrub-weekly@.timer
[Unit]
Description=Weekly zpool scrub timer for %i
Documentation=man:zpool-scrub(8)

[Timer]
OnCalendar=weekly
Persistent=true
RandomizedDelaySec=1h
Unit=zfs-scrub@%i.service

[Install]
WantedBy=timers.target
```


And it have a pairing templated service 
```
% cat /lib/systemd/system/zfs-scrub@.service
[Unit]
Description=zpool scrub on %i
Documentation=man:zpool-scrub(8)
Requires=zfs.target
After=zfs.target
ConditionACPower=true
ConditionPathIsDirectory=/sys/module/zfs

[Service]
EnvironmentFile=-/etc/default/zfs
ExecStart=/bin/sh -c '\
if /usr/sbin/zpool status %i | grep -q "scrub in progress"; then\
exec /usr/sbin/zpool wait -t scrub %i;\
else exec /usr/sbin/zpool scrub -w %i; fi'
ExecStop=-/bin/sh -c '/usr/sbin/zpool scrub -p %i 2>/dev/null || true'
```

The weekly default for systemd is monday morning. 