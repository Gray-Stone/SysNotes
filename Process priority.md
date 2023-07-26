# Priority Adjustment

## Set priority

Priority is not the same as nice value on Linux. 

https://stackoverflow.com/a/52501811

This answer explained things very well on what does a priority value, and PR value are related. (PR is what htop shows)

Basically using chrt will set the priority (not PR), and higher value set through chrt, the more priority is given, Setting 99 will result in a PR of `rt` which might not be good as it interfere with systems. 

example 

```
sudo chrt -rr -p 50 "$(pgrep my_process_name)"
```

Since running process under sudo might be a bad idea, the above command sets the priority after the process is launched. chrt takes the PID value pgrep is just make it convenient to do PIDs. 

Value of 50 will put this process in front of most other process on the system.


## CPU isolation 

https://unix.stackexchange.com/questions/326579/how-to-ensure-exclusive-cpu-availability-for-a-running-process

The answer includes good examples on isolating a core and set specific task onto it. The one I get to work on Ubuntu is to set through grub https://unix.stackexchange.com/a/326585

This isolated cpu through this grep setting (which uses isolcpu) is only use-able when using `taskset` to specifically put process on it.

There is another answer about setting it through `cpuset`. However, seems like there are something about this being deprecated on newer system and need to setup through systemd. 

## Pin process onto cpu

`taskset` is used to pin process onto a cpu. This doesn't require root priority.

`taskset -c 3 <a command to run | or PID>` This is to run command with cpu 3 or set already launch PID to cpu 3. 

Without the `-c`, taskset takes a cup mask, which might be more complicated to use.  