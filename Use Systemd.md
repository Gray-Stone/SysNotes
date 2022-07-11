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

A note from the (accept manual entry)[https://www.freedesktop.org/software/systemd/man/systemd.socket.html#Accept=]
>For IPv4 and IPv6 connections, the REMOTE_ADDR environment variable will contain the remote IP address, and REMOTE_PORT will contain the remote port. This is the same as the format used by CGI. For SOCK_RAW, the port is the IP protocol.

man page https://www.man7.org/linux/man-pages/man5/systemd.socket.5.html

Passing of the socket have two mode (within `Accept=yes`). Pass down the fd of the socket, program need to use `sd_listen_fds()` to check for it. Or use `inetd` style where stdin and stdout is used. 

### C++ executable as the socket unit's target

These steps are needed for using `sd_listen_fds()` to check how many sockets are passed, and use `SD_LISTEN_FDS_START` to know the starting file descriptor:

`#include <systemd/sd-daemon.h>` this library is needed, which is installed from `sudo apt install libsystemd-dev`. In addition, in CMakeList, `target_link_libraries( ${TARGET} -lsystemd` this `-lsystemd` item is needed for linker to find the library. 

However, when running in `Accept=yes` mode, these are not necessary. You will always only get 1 file descriptor. Also the `SD_LISTEN_FDS_START` is actually hard coded to 3 `#define SD_LISTEN_FDS_START 3`. This mean if you know what you are doing. you can skip all these and just use file descriptor 3.  

### Wrapper for the actual socket unit executable.

The executable within socket unit's template unit could actually have wrappers (like shell scripts)

The file descriptor will simply kept working when the executable is run as a sub-process (I need to look into why)

In this wrapper script you can do many things including put on tshark. However, for filtering to work nicely, we might also want all the address info.

The `REMOTE_ADDR` and `REMOTE_PORT` is defined. However there are nothing for local address info, you have to fetch it from file descriptor

If the wrapper is a shell script and Python3 is installed. Then a quick inline python trick could be used. 

``` bash 
#! /usr/bin/bash
python_find_local(){
python3 <<EOF
import socket
local_addr , local_port , *_ = socket.socket(fileno=3).getsockname()
print(f"""
LOCAL_ADDR={local_addr}
LOCAL_PORT={local_port}
""")
EOF
}

eval $(python_find_local)

echo "local addr is ${LOCAL_ADDR}"
echo "local port is ${LOCAL_PORT}"

# Then call your exec file 
```

### Instance name %i

In a systemd template unit, the %i will be expended to the template name (which is between the `@` and `.service` part.)

However, when you use `systemctl` or `journalctl` this unit show up as `xxx@<num>-<local.addr>-<remote.addr>`, but the %i is only a number. (this might change for different version)

This mean the shell script in the templated unit can't call journalctl get its own log.

The work-around is to do `journalctl -u "xxx_unit@${INSTANCE}-${LOCAL_ADDR}:${LOCAL_PORT}-${REMOTE_ADDR}:${REMOTE_PORT}"` where `${INSTANCE}` is the %i passed in as command line arg 


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

`systemctl edit <unit_name>` will create the correct folder, file and open the editor pointing to these files for you. (it can even edit multiple units at the same time). Using this command, adding override content becomes much easier. To use this command in a script that auto inject override contents requires some special trick: 

```
echo -e "overriding content you want" | sudo SYSTEMD_EDITOR=tee systemctl edit <your_unit> <other_unit_can_be_edited_together>
```

`tee -a` also work if you only want to append to existing overload file


---

# Transient unit

https://www.commandlinux.com/man-page/man1/systemd-run.1.html

`systemd-run` is the way to quickly run a command as a systemd unit without setting up the unit file. The command will be put into a systemd transient unit to run.

The transient unit will have a random name if not assigned, and will disappear after it is finished (not auto complete-able from systemctl command)

The following script can help handle this, also launched `journalctl` on this unit to make it as just running a normal command. 

``` bash
NC=$'\e[0m' ; RED=$'\e[0;31m' ; GREEN=$'\e[0;32m'

function sys-run () {
  set -xv
  echo "$RED running $GREEN $1 $RED with $GREEN ${@:2} $NC"
  UNIT_NAME="$1_$(date +%b%d_%H:%M:%S)"
  systemd-run --user --unit "${UNIT_NAME}"  --remain-after-exit    /bin/zsh -c " source ~/.zshrc ; ${*} "
  journalctl --user -fu ${UNIT_NAME}
}
```

