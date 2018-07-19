# -*- mode: ruby -*-
Vagrant.configure("2") do |config|

  # shared folders
  # config.vm.synced_folder "web", "/srv/www"

  # SSH keys
  ssh_pub_key = File.readlines("#{Dir.home}/.ssh/id_rsa.pub").first.strip
  config.vm.provision 'shell', inline: 'mkdir -p /root/.ssh'
  config.vm.provision 'shell', inline: "echo #{ssh_pub_key} >> /root/.ssh/authorized_keys"
  config.vm.provision 'shell', inline: "echo #{ssh_pub_key} >> /home/vagrant/.ssh/authorized_keys", privileged: false

  # swap file
  # config.vm.provision 'shell', inline: 'dd if=/dev/zero of=/swapfile bs=1024 count=1048576'
  # config.vm.provision 'shell', inline: 'mkswap /swapfile'
  # config.vm.provision 'shell', inline: 'swapon -a'

  # vbox os
  # debian: stretch64|jessie64|wheezy64
  # ubuntu: bionic64|xenial64|trusty64|precise64
  # centos: 7|6
  # fedora: 28-atomic-host|27-atomic-host|26-atomic-host
  vbox_repo = "debian"
  vbox_os   = "stretch64"
  vbox_cpu  = 1
  vbox_mem  = "1024"
  vbox_ip   = "192.168.8"

  (51..53).each do |i|
    config.vm.define "#{vbox_os}-#{i}", primary: true, autostart: true do |node|
      node.vm.box = "#{vbox_repo}/#{vbox_os}"
      node.vm.network "public_network", ip: "#{vbox_ip}.#{i}", bridge: "wlan0"
      node.vm.hostname = "vm-#{i}"

      node.vm.provider "virtualbox" do |v|
        v.name = "ansible-#{vbox_os}-#{i}"
        v.cpus = "#{vbox_cpu}"
        v.memory = "#{vbox_mem}"

        # audio/video
        v.customize ["modifyvm", :id, "--audio", "none"]
        v.customize ["modifyvm", :id, "--vram", "16"]

        # cpu limit
        v.customize ["modifyvm", :id, "--cpuexecutioncap", "80"]

        # SATA: later :)
        # v.customize ["storagectl", :id, "--name", "IDE Controller", "--remove"]
        # v.customize ["storagectl", :id, "--name", "SATA Controller", "--add", "sata", '--portcount', 4]
        end
      end
  end
end
