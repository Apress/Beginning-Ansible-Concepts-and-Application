# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure("2") do |config|
  config.vm.box = "ubuntu/focal64"

  # globally disable the default synced folder
  config.vm.synced_folder ".", "/vagrant", disabled: true

  # do not create a secure private key per host, we want to use a single key
  # this allows the controller to connect to the target hosts
  config.ssh.insert_key = false

  # Set provider virtualbox to use GUI, for two reasons:
  #  1. More obvious to the user what is running, so they don't consume too much background resource
  #  2. To work around an x64 boot issue in Virtualbox when hardware virtualization is disabled in BIOS
  config.vm.provider "virtualbox" do |v|
#    v.gui = true
    v.cpus = 1
  end

  # The Ansible controller machine
  config.vm.define :controller, primary: true do |controller|
    controller.vm.hostname = "ansible-controller"
    controller.vm.network "private_network", ip: "192.168.98.100"

    # Sync the local directory with ansible-friendly permissions
    controller.vm.synced_folder ".", "/vagrant",
      owner: "vagrant",
      mount_options: ["dmode=755,fmode=644"]

    controller.vm.provision "file", source: "~/.vagrant.d/insecure_private_key", destination: "$HOME/.ssh/id_rsa"

    controller.vm.provision "shell", inline: <<-SHELL
      chmod 600 /home/vagrant/.ssh/id_rsa
      apt-get install -y ansible sshpass
    SHELL
  end


  # Example web servers
  (1..2).each do |i|
    config.vm.define "web-00#{i}" do |node|
      node.vm.hostname = "web-00#{i}"
      node.vm.network "private_network", ip: "192.168.98.11#{i}"
    end
  end

  # Example load balancer
  (1..1).each do |i|
    config.vm.define "lb-00#{i}" do |node|
      node.vm.hostname = "lb-00#{i}"
      node.vm.network "private_network", ip: "192.168.98.12#{i}"
    end
  end

  # Example database server
  (1..1).each do |i|
    config.vm.define "db-00#{i}" do |node|
      node.vm.hostname = "db-00#{i}"
      node.vm.network "private_network", ip: "192.168.98.13#{i}"
    end
  end

  # Generic provisioner to ensure we have python available
  # This is an ansible requirement for all managed nodes
  config.vm.provision "shell", inline: <<-SHELL
    # Disable hardware based sha256sum in apt, not ideal - but Windows 10 WSL breaks VirtualBox
    # and we can't really ask everyone to disable that
    # See: https://askubuntu.com/questions/1235914/hash-sum-mismatch-error-due-to-identical-sha1-and-md5-but-different-sha256/1241893
    mkdir /etc/gcrypt
    echo all >> /etc/gcrypt/hwf.deny

    apt-get update
    apt-get install -y python3-minimal python3-apt avahi-daemon tree

    rm -f /etc/update-motd.d/*
    sed -i "s/^ENABLED=.*/ENABLED=0/" /etc/default/motd-news
  SHELL

  config.vm.boot_timeout = 360
end
