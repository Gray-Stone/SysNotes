# Win-app on Linux

Run windows apps in linux VM, while mapping them out to the linux using RDP-app mode (similar to x11 windows over network)


This is the currently maintained repo for this project. 
https://github.com/winapps-org/winapps

## First tune the KVM.

https://github.com/winapps-org/winapps/blob/main/docs/KVM.md

Some of the edits require changing the xml. A preference in virt-manager need to be enabled to allowed editing xml.


One of the step asked for changing clock and cpu time to reduce idle cpu usage. 

Suggested change:
```
<clock offset='localtime'>
  <timer name='hpet' present='yes'/>
  <timer name='hypervclock' present='yes'/>
</clock>
```

In my existing windows vm, the hpet is off. at idle, qemu cpu usage is around 150%, enabling this, usage is around 100 - 150 after closing the window for a while. The setting change does have some benefit, but not that big a change. 



## Setup host side 

The repo itself needs to be cloned. and some dependencies are required.

```
sudo apt install -y freerdp2-x11
```

```
git clone https://github.com/winapps-org/winapps.git
```


## Running it. 

When WinApps trying to remote into the machine and use windows. It will create a "full" screen RDP session. then later on pop windows out. If the same windows is already logged in from somewhere else. Then you will be stuck for a 20sec ish timeout so windows give other logged-in user a chance to reject you taking over the control (since only one login at time)