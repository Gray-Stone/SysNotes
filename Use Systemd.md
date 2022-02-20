# Systemd Unit

## General info

Use systemd to manage process and services for you. Systemd manage things by unit. Each unit is mostly an executable.

[Blog post from digital ocean for basic info of systemd unit.](https://www.digitalocean.com/community/tutorials/understanding-systemd-units-and-unit-files)

---

## User unit

User unit is a way to have systemd unit running under a (current) user. This mean no need for sudo. 

An [article from Arch Linux](https://wiki.archlinux.org/title/Systemd/User) about user unit.

Put the unit file under `{HOME}/.config/systemd/user` and `systemctl --user daemon-reload` would be able to detect it. 
Use `systemctl --user` to control a user unit, which is similar to a normal unit but run under current user. 

Normally this unit will only run when user is logged in. But can also stay on using `loginctl enable-linger`

Simplest unit:

```
[Unit]
Description=What is this? 
[Service]
ExecStart=echo "hello from this unit"
```

## No Instal unit

`[Install]` section in a unit file describe the dependency of this unit (and thus how it will start/stop related to other units) 

When a unit doesn't have a Install, it can't be enabled. This mean it won't auto start, but instead you can manually start it. 

https://stackoverflow.com/questions/52287316/systemd-an-service-unit-without-install-section-will-it-be-auto-run-on-boo


## Multiple Executables

One unit mostly only take 1 executable. There are few ways to run more then one executable

Use `/bin/bash -c " <bash script issue multi command and optionally throw some to background> "`. When using this option, you might want to use `Type=forking` and `RemainAfterExit=yes` to prevent systemd lost track of background task.

Use multiple unit and use dependency. A unit will be the overall trigger, use `WantedBy` in each executable's individual unit so the trigger unit will start them. Also need to use `PartOf` so each individual executable can be stopped when trigger unit stops.

https://stackoverflow.com/questions/48195340/systemd-with-multiple-execstart


## Socket Units

Let systemd create the socket and listen to it. Only turn on the actual executable when connection come in and systemd will pass down the socket. 

The `Accept=yes` option seems to affect a huge difference on executable use-case. Should be chosen carefully. 

man page https://www.man7.org/linux/man-pages/man5/systemd.socket.5.html

Passing of the socket have two mode (within `Accept=yes`). Pass down the fd of the socket, program need to use `sd_listen_fds()` to check for it. Or use `inetd` style where stdin and stdout is used. 