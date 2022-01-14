# -*- mode: ruby -*-
# vi: set ft=ruby :

# The base of the the hostname
HOSTNAME_BASE = "archboot"

# The TLD domain the hosts belong to
DOMAIN_NAME = "test.lan"

# All Vagrant configuration is done below. The "2" in Vagrant.configure
# configures the configuration version (we support older styles for
# backwards compatibility). Please don't change it unless you know what
# you're doing.
Vagrant.configure("2") do |config|
  # The most common configuration options are documented and commented below.
  # For a complete reference, please see the online documentation at
  # https://docs.vagrantup.com.

  # Every Vagrant development environment requires a box. You can search for
  # boxes at https://vagrantcloud.com/search.
  # Boot the normal ubuntu for Aarch64
  config.vm.box = "fredleb/ubuntu2004-server-aarch64"

  # ARM + Ubuntu on a x86_64 machine is not fast when trying to spwan loads of
  # machines at once. So give it a bit more time than the default 5 min.
  config.vm.boot_timeout = 600 # 10 mins

  # Share the local directory to guest's /vagrant
  #config.vm.synced_folder "./guest/", "/vagrant", type: "nfs"
  config.vm.synced_folder "./guest/", "/vagrant", type: "rsync"

  # Create several machines
  name = "#{HOSTNAME_BASE}"
  config.vm.define "#{name}" do |server|
    server.vm.hostname = "#{name}.#{DOMAIN_NAME}"

    ## Qemu EFI must be installed on the machine for the AArch64 target:
    # Ubuntu: sudo apt install qemu-efi qemu-efi-aarch64 qemu-system-arm qemu-utils
    server.vm.provider :libvirt do |libvirt|
      libvirt.driver = "qemu"
      libvirt.storage_pool_name = "big_pool"
      libvirt.memory = 4096
      libvirt.cpus = 4
      libvirt.machine_type = "virt"
      libvirt.machine_arch = "aarch64"
      libvirt.cpu_mode = "custom"
      libvirt.cpu_model = "cortex-a72"
      libvirt.graphics_type = "none"
      libvirt.features = ['acpi',  'gic version=\'2\'']
      libvirt.loader = "/usr/share/AAVMF/AAVMF_CODE.fd"
      libvirt.nvram = File.expand_path("./nvram/nvram.fd")

      ## Add the target disk image.
      # The file itself should not exist before "vagrant up" is called !
      # It will show up as /dev/vdb
      libvirt.storage :file, :size => '25G', :path => "archboot_target.qcow2"
    end
  end

  config.vm.provision "shell", name: "Disable ubuntu updates",
    inline: <<-SHELL
      systemctl stop unattended-upgrades || true
    SHELL

  config.vm.provision "shell", name: "Prepare the target disk",
    inline: <<-SHELL
      sfdisk /dev/vdb < /vagrant/vdb.sfdisk
      mkfs.fat -F 32 /dev/vdb1
      mkfs.ext4 /dev/vdb2
      e2label /dev/vdb2 arch_os
    SHELL

  config.vm.provision "shell", name: "Mount the target disk",
    inline: <<-SHELL
      mount /dev/vdb2 /mnt
      mkdir -p /mnt/boot
      mount /dev/vdb1 /mnt/boot
    SHELL

  config.vm.provision "shell", name: "Add target swap file",
    inline: <<-SHELL
      fallocate -l 4G /mnt/swapfile
      chmod 0600 /mnt/swapfile
      mkswap /mnt/swapfile
      swapoff /swap.img
      swapon /mnt/swapfile
    SHELL

  config.vm.provision "shell", name: "Install Arch installation tools",
    inline: <<-SHELL
      apt-get install -y libarchive-tools arch-install-scripts
    SHELL

  config.vm.provision "shell", name: "Untar the archive (this can take a while...)",
    inline: <<-SHELL
      bsdtar -xpf /vagrant/tmp/ArchLinuxARM-aarch64-latest.tar.gz -C /mnt
    SHELL

  config.vm.provision "shell", name: "Generate target fstab",
    inline: <<-SHELL
      genfstab -U /mnt >> /mnt/etc/fstab
    SHELL

  config.vm.provision "shell", name: "Arch: initialize pacman",
    inline: <<-SHELL
      arch-chroot /mnt pacman-key --init
      arch-chroot /mnt pacman-key --populate archlinuxarm
      arch-chroot /mnt pacman -Syy
    SHELL

  config.vm.provision "shell", name: "Arch: prepare bootloader",
    inline: <<-SHELL
      cp --recursive --no-preserve=mode,ownership /vagrant/boot/loader /mnt/boot/loader
      arch-chroot /mnt bootctl install
      arch-chroot /mnt bootctl update
    SHELL

  # The Arch linux VM could now boot on its own.
  # Let's make it Vagrant compatible.

  config.vm.provision "shell", name: "Arch: install sudo",
    inline: <<-SHELL
      arch-chroot /mnt pacman --needed --noconfirm -S sudo
    SHELL

  config.vm.provision "shell", name: "Arch: set root password to \"vagrant\"",
    inline: <<-SHELL
      arch-chroot /mnt echo "root:vagrant" | arch-chroot /mnt chpasswd
    SHELL

  config.vm.provision "shell", name: "Arch: add \"vagrant\" user",
    inline: <<-SHELL
      arch-chroot /mnt useradd -m vagrant
      arch-chroot /mnt echo "vagrant:vagrant" | arch-chroot /mnt chpasswd
      cp --recursive /vagrant/home /mnt/
      arch-chroot /mnt chown -R vagrant:vagrant /home/vagrant
      arch-chroot /mnt chmod -R 0600 /home/vagrant/.ssh
      arch-chroot /mnt chmod 0700 /home/vagrant/.ssh
      echo "vagrant ALL=(ALL) NOPASSWD: ALL" > /mnt/etc/sudoers.d/vagrant
    SHELL

  config.vm.provision "shell", name: "Arch: remove Arch's \"alarm\" user",
    inline: <<-SHELL
      arch-chroot /mnt userdel -r alarm
    SHELL

  config.vm.provision "shell", name: "Arch: install AUR package helper",
    inline: <<-SHELL
      arch-chroot /mnt pacman --needed --noconfirm -S git base-devel go

      arch-chroot /mnt sudo -u vagrant bash -xc "\
        cd ~vagrant && \
        git clone https://aur.archlinux.org/yay.git && \
        pushd yay && \
        makepkg -si --noconfirm && \
        popd && \
        rm -rf yay"
    SHELL

  config.vm.provision "shell", name: "Arch: empty pacman's cache",
    inline: <<-SHELL
      arch-chroot /mnt pacman -Scc --noconfirm
    SHELL

  config.vm.provision "shell", name: "Arch: blank machine-id",
    inline: <<-SHELL
      truncate -s 0 /mnt/etc/machine-id
    SHELL

end
