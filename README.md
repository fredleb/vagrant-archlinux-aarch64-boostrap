# vagrant-archlinux-aarch64-boostrap
Vagrantfile and doc to bootstrap a arch linux image on Aarch64

# Assumptions

I have setup libvirt to use a conveniently located storage pool "big_pool" for my VMs. You might need to change that.

# Usage

## Get the latest Arch Linux image for Aarch64

Go to https://archlinuxarm.org/platforms/armv8/generic and download ArchLinuxARM-aarch64-latest.tar.gz to *guest/tmp*

## Vagrant up !

Will build the target image !

Once done, login to the VM.

### DHCP client-id

The clinet-id used by the DHCP to identify each client is normally derived (as it should be) from the machine's ID contained in _/etc/machine-id_.

Before we spawn several clones of this installation, we need to empty it (**NOT DELETE IT**). It will be regenerated at the next boot.

```
sudo truncate -s 0 /mnt/etc/machine-id
```

Now you can shut down the machine cleanly.

### Starting the new VM

Then use the "clone" of the Virtual Machine Manager:
 - do NOT clone the ubuntu disk image
 - do clone the Arch taget dick image
 - then open the description of the new cloned VM and remove the first disk altogether

Go to your pool and delete the cloned nvram variables file: something is wrong with it somehow...

Now you can start the VM and it enjoy far better speed than with the Ubuntu 20.04 server image...

# Box it

Remember to clear the machine ID !

## Getting the disk image

Copy the target dick to a local sparse file:
```
qemu-img convert -O qcow2 /home/vm/big_pool/archboot_target.qcow2 ./box.img
```

## Packing

I prepare my own Vagrantfile (see below) and metdata.json.

Then I pack them together in a box file:
```
tar cvzf archlinux-aarch64.box ./metadata.json ./info.json ./Vagrantfile ./box.img
```
or faster:
```
tar cv -S --totals ./metadata.json ./info.json ./Vagrantfile ./box.img | pigz -c > archlinux-aarch64.box
```

And added the box to my local vagrant:
```
vagrant box add archlinux-aarch64.box --name archlinux-aarch64
```
