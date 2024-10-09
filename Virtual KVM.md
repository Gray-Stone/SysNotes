

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

