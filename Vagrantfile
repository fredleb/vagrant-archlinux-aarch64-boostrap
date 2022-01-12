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



end
