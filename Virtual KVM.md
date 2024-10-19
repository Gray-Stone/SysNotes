

Run `kvm-ok` for system support 


The base of vm stuff

```sh
sudo apt install libvirt-clients libvirt-daemon qemu qemu-kvm
```

For GUI management interface:
```sh
sudo apt install virt-manager
```

To get ability for briding the vm out and some network benefits
```sh 
sudo apt install bridge-utils
```


## Folder sharing Via libvirt to windows 

https://www.debugpoint.com/kvm-share-folder-windows-guest/



## Vagrant 

Install Vagrant 
```sh
wget -O- https://apt.releases.hashicorp.com/gpg | sudo gpg --dearmor -o /usr/share/keyrings/hashicorp-archive-keyring.gpg
echo "deb [signed-by=/usr/share/keyrings/hashicorp-archive-keyring.gpg] https://apt.releases.hashicorp.com $(lsb_release -cs) main" | sudo tee /etc/apt/sources.list.d/hashicorp.list
sudo apt update && sudo apt install vagrant
```


### Setting up Vagrant with libvirt

Vagrant default uses virtual box as backend. We need the vagrant-libvirt plugin for it. https://github.com/vagrant-libvirt/vagrant-libvirt Documentation under https://vagrant-libvirt.github.io/vagrant-libvirt/

Following the document, these packages are needed as well 
```sh
sudo apt install ebtables libguestfs-tools ruby-fog-libvirt libvirt-dev
```

ebtables is for adding ethernet rules, like intercept package etc. 
libguestfs-tools is for access and modifying VM disk images. 
libvirt-dev is build dependency for vagrant-libvirt plugin.

Also want to make sure the vagrant-libvirt is installed via vagrant
```
sudo apt-get purge vagrant-libvirt
sudo apt-mark hold vagrant-libvirt
```

Install the plugin 
```
vagrant plugin install vagrant-libvirt
```

The default provider is virtualbox. To set libvirt as provider: `vagrant up --provider=libvirt` or `export VAGRANT_DEFAULT_PROVIDER=libvirt` 


## Libvirt Windows 

Some tools are needed to be installed to allow seamless cursor transition, and mapping outside folder into windows. 


https://serverfault.com/a/457611

https://www.spice-space.org/download.html -> Guest section



I am using this LTSC version of winbox which supports libvirt (but not the latest version. The default config needs slight change to not pinned at latest version)
https://portal.cloud.hashicorp.com/vagrant/discover/it-gro/win10-ltsc-eval

## Winapp

The Magical app that use RDP to make app inside vm showup like native Linux app.

Libvirt setup: 

Require this URI setting.
```sh 
echo "export LIBVIRT_DEFAULT_URI=\"qemu:///system\"" >> ~/.bashrc
``` 

Require `qemu-guest-agent` package and systemd unit to be running


Add user to group `kvm` and `libvirt` so operation on vm can be done without being root.


### Virt IO driver 

Winapp ask for the install of virt IO driver. This is a common driver that is mentioned in other tutorials. 

This link is directly going to the latest version by redhat.
https://fedorapeople.org/groups/virt/virtio-win/direct-downloads/latest-virtio/virtio-win.iso


### Config the VM for winapp

There are a huge list of configs to go through before actually using the VM.

https://github.com/winapps-org/winapps/blob/main/docs/libvirt.md#creating-a-windows-vm

note: 
* Under step 13 of the above linked section, the line `<source mode='bind'/>` is removed by virt-manager as soon as added and applyed. Will just ignore this small difference for now.

