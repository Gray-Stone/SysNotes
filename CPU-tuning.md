# Playing with CPU 



## Monitor the frequency of each core in terminal 

https://unix.stackexchange.com/a/265611

```
watch -n.1 "grep \"^[c]pu MHz\" /proc/cpuinfo"
```

```
Every 0.1s: grep "^[c]pu MHz" /proc/cpuinfo                                                                                                                GAT-X103: Thu Jan 18 14:49:02 2024

cpu MHz         : 1015.051
cpu MHz         : 922.648
cpu MHz         : 2400.000
cpu MHz         : 1186.398
cpu MHz         : 2400.000
cpu MHz         : 1163.815
cpu MHz         : 2400.000
cpu MHz         : 2400.000
cpu MHz         : 1056.894
cpu MHz         : 2400.000
cpu MHz         : 799.526
cpu MHz         : 2400.000
cpu MHz         : 975.742
cpu MHz         : 2400.000
cpu MHz         : 1190.422
cpu MHz         : 1095.141
cpu MHz         : 875.382
cpu MHz         : 2400.000
cpu MHz         : 2400.000
```

## Manually set cpu frequency

cpuf bash script 

or 

cpupower-info

https://askubuntu.com/questions/1141605/gui-or-simple-bash-script-to-throttle-the-cpu

