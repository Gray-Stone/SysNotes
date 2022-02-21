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

Normally this unit will only run when user is logged in. But can also stay on using `loginctl enable-linger`. Can use the loginctl status `loginctl user-status` to check

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


## Stop your unit in expected way

Man page:
https://www.freedesktop.org/software/systemd/man/systemd.kill.html#


`KillMode=` decide how systemd track the processes within a unit. 
* `KillMode=control-group` will track and send signal to all process within the unit. 
* `KillMode=mixed` will send terminate signal to main process, and subsequent kill signal is sent to all remaining process. 
* `KillMode=process` only kill the main process (not recommended)
* `killMode=none` No process is killed (strongly not recommended)

You can also use `ExitType` to tell systemd how to consider when a unit is terminated (by main process or all process must finish). This is available at version 250 (not on ubuntu20)

When stopping a unit, by default there are two steps.:
quote from man page
> Processes will first be terminated via SIGTERM (unless the signal to send is changed via KillSignal= or RestartKillSignal=). Optionally, this is immediately followed by a SIGHUP (if enabled with SendSIGHUP=). If processes still remain after the main process of a unit has exited or the delay configured via the TimeoutStopSec= has passed, the termination request is repeated with the SIGKILL signal or the signal specified via FinalKillSignal= (unless this is disabled via the SendSIGKILL= option). 

Use `killSignal=` to set the signal used on first step. and `FinalKillSignal=` to set the second signal to use. 

`SendSIGKILL=` is the on-off switch for the second stopping signal. It takes a boolean value and default to yes. When turned off, the unit will not restart until tracked process has actually died.

`RestartKillSignal=` let you set a different signal from the one specified in `killSignal` when unit restarts.

`TimeoutStopSec=` will set the time between `SIGTERM` and `SIGKILL` when `ExecStop=` is not set. When `ExecStop=` is set, this is also the timeout for running each `ExecStop=` command

`TimeoutAbortSec=` generally used to give a unit more time to write to core file after terminate. for detail [see man page](https://www.freedesktop.org/software/systemd/man/systemd.service.html#TimeoutAbortSec=)

`TimeoutSec=` is a shorthand for configuring both `TimeoutStartSec=` and `TimeoutStopSec=` to the specified value.  

Example for setting terminate signal for a user unit that usually expect sigint.
```
[Service]
KillMode=control-group
killSignal=SIGINT
FinalKillSignal=SIGKILL
TimeoutStopSec=10
```
---


## Some useful settings:

To turn off log limit (doesn't work in 1804):
```
LogRateLimitIntervalSec=0
LogRateLimitBurst=0
```

Make a quick user unit and run command like you would in terminal:
```
[Service]
ExecStart=/bin/zsh -c "source ~/.zshrc ; sleep 2 ; ${USE_MY_TERMINAL} "
```

---

## Work with other scripts (like bash):

#### Detect a unit
To check if a systemd unit exists:
```
if ! systemctl cat ${UNIT_I_WANT} >/dev/null 2>&1; then
	echo "Unit ${UNIT_I_WANT} doesn't exists
else
	echo "Unit ${UNIT_I_WANT} exists
fi
```

#### Override

Unit can have override files that add/replace entries to the system file. `${path_to_unit_file}.d/override.conf` Is the override file. Needs do daemon-reload after making this file.

