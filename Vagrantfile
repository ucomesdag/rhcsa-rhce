# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure(2) do |config|

  # Path where the extra disks should be stored when using VirtualBox
  vbox_vm_path = "" # VirtualBox ONLY!

  # Storage pool where the extra disks should be stored when using libVirt
  libvirt_storage_pool = "" # libVirt ONLY!

  # Red Hat subscription credentials if you choose to use a RHEL box.
  rh_user   = ""
  rh_passwd = ""

  # Memory configuration
  classroom_memory = 2048 # Recommended: 2048 MiB / Minimum: 512 MiB
  server_memory = 1024 # Recommended: 1024 MiB / Minimum: 512 MiB
  desktop_memory = 1024 # Recommended: 1024 MiB / Minimum: 512 MiB
  extra_disk_size = 2 # Recommended: 10 GiB / Minimum: 2 GiB

  # VM guest configuration
  config.vm.box = "rhel/7.2" # Use centos/7 or the label you used when adding the downloaded rhel box, for expl: rhel/7.2

  # Don't modify beyond here
  conname = "Wired connection 1"
  devname = "eth1"
  if config.vm.box =~ /^rhel\/.*$/
    conname = "eth1"
    devname = "eth1"
    if Vagrant.has_plugin?('vagrant-registration')
      now = Time.new
      config.registration.name = "rhel-lab-" + now.strftime("%Y%m%d%H%M")
      if rh_user == "" or rh_passwd == ""
        config.registration.username = ENV['SUBSCRIPTION_USERNAME']
        config.registration.password = ENV['SUBSCRIPTION_PASSWORD']
      else
        config.registration.username = rh_user
        config.registration.password = rh_passwd
      end
      if ! config.registration.username or ! config.registration.password
        puts "+-----------------------------------------------------------------------------+"
        puts "|    The VM guest is set to a RHEL box, to continue a Red Hat subscription    |"
        puts "|                               is required !!!                               |"
        puts "+-----------------------------------------------------------------------------+"
        puts
        puts " You can use the environment variables 'SUBSCRIPTION_USERNAME' and"
        puts " 'SUBSCRIPTION_PASSWORD' to set your credentials."
        puts
        puts " Example:"
        puts "   export SUBSCRIPTION_USERNAME='UserName' SUBSCRIPTION_PASSWORD='Password'"
        puts
        puts
        exit 1
      end
    end
  end

  config.vm.provider "virtualbox" do |vbox, override|
    # vbox.gui = true
    if vbox_vm_path == ""
      vbox_vm_path = ENV['VBOX_VM_PATH']
    end
  end

  config.vm.provider "libvirt" do |libvirt, override|
    if libvirt_storage_pool == ""
      libvirt_storage_pool = ENV['LIBVIRT_STORAGE_POOL']
    end
    libvirt.driver = "kvm"
    libvirt.storage_pool_name = libvirt_storage_pool
  end

  config.vm.provision :shell, path: "scripts/default-provision"

  config.vm.define :classroom do |classroom_config|
    classroom_config.vm.hostname = "classroom.example.com"
    classroom_config.vm.network "private_network", ip: "172.25.0.254", auto_config: false
    classroom_config.vm.provision :shell, run: "always", inline: "(nmcli device connect '#{devname}' &) && sleep 10 && nmcli con modify '#{conname}' ipv4.addresses 172.25.0.254/24 ipv4.dns 172.25.0.254,8.8.8.8 ipv4.route-metric 10 ipv4.method manual && nmcli con up '#{conname}'"
    classroom_config.vm.provision :shell, path: "scripts/classroom-provision"
    classroom_config.vm.provider "virtualbox" do |vbox, override|
      vbox.cpus = 1
      vbox.memory = classroom_memory
    end
    classroom_config.vm.provider "libvirt" do |libvirt, override|
      libvirt.cpus   = 1
      libvirt.memory = classroom_memory
    end
  end

  config.vm.define :server do |server_config|
    server_config.vm.hostname = "server.example.com"
    server_config.vm.network "private_network", ip: "172.25.0.11", auto_config: false
    server_config.vm.provision :shell, run: "always", inline: "(nmcli device connect '#{devname}' &) && sleep 10 && nmcli con modify '#{conname}' ipv4.addresses 172.25.0.11/24 ipv4.gateway 172.25.0.254 ipv4.dns 172.25.0.254 ipv4.route-metric 10 ipv4.method manual && nmcli con up '#{conname}'"
    server_config.vm.network "private_network", ip: "172.25.0.12", auto_config: false
    server_config.vm.provision :shell, path: "scripts/server-provision"
    server_config.vm.provider "virtualbox" do |vbox, override|
      vbox.cpus = 1
      vbox.memory = server_memory
      if !File.exist?(vbox_vm_path + 'rhel_server_2.vdi')
        vbox.customize ['createhd', '--filename', vbox_vm_path + 'rhel_server_2.vdi', '--variant', 'Fixed', '--size', extra_disk_size * 1024]
      end
      vbox.customize ['storageattach', :id,  '--storagectl', 'IDE', '--port', 1, '--device', 0, '--type', 'hdd', '--medium', vbox_vm_path + 'rhel_server_2.vdi']
    end
    server_config.vm.provider "libvirt" do |libvirt, override|
      libvirt.cpus   = 1
      libvirt.memory = server_memory
      libvirt.storage :file, :size => extra_disk_size.to_s + 'G'
    end
  end

  config.vm.define :desktop do |desktop_config|
    desktop_config.vm.hostname = "desktop.example.com"
    desktop_config.vm.network "private_network", ip: "172.25.0.10", auto_config: false
    desktop_config.vm.provision :shell, run: "always", inline: "(nmcli device connect '#{devname}' &) && sleep 10 && nmcli con modify '#{conname}' ipv4.addresses 172.25.0.10/24 ipv4.gateway 172.25.0.254 ipv4.dns 172.25.0.254 ipv4.route-metric 10 ipv4.method manual && nmcli con up '#{conname}'"
    desktop_config.vm.provision :shell, path: "scripts/desktop-provision"
    desktop_config.vm.provider "virtualbox" do |vbox, override|
      vbox.cpus = 1
      vbox.memory = desktop_memory
      if !File.exist?(vbox_vm_path + 'rhel_desktop_2.vdi')
        vbox.customize ['createhd', '--filename', vbox_vm_path + 'rhel_desktop_2.vdi', '--variant', 'Fixed', '--size', extra_disk_size * 1024]
      end
      vbox.customize ['storageattach', :id,  '--storagectl', 'IDE', '--port', 1, '--device', 0, '--type', 'hdd', '--medium', vbox_vm_path + 'rhel_desktop_2.vdi']
    end
    desktop_config.vm.provider "libvirt" do |libvirt, override|
      libvirt.cpus   = 1
      libvirt.memory = desktop_memory
      libvirt.storage :file, :size => extra_disk_size.to_s + 'G'
    end
  end

end
