# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure(2) do |config|

  config.vm.box = "centos/7"

  config.vm.hostname = "dev-shangrila"

  if Vagrant::Util::Platform.windows?
    ENV["VAGRANT_DETECTED_OS"] = ENV["VAGRANT_DETECTED_OS"].to_s + " cygwin"
  end

  # Create a forwarded port mapping which allows access to a specific port
  # within the machine from a port on the host machine. In the example below,
  # accessing "localhost:8080" will access port 80 on the guest machine.
  # config.vm.network "forwarded_port", guest: 80, host: 8080

  config.vm.network "private_network", ip: "192.168.33.10"

  # If you are using Windows, it may be more faster and to use the RSync.
  # type: "rsync"
  config.vm.synced_folder "ansible", "/home/vagrant/ansible",
    :owner => 'vagrant', :group => 'vagrant',
    :mount_options => ['dmode=755', 'fmode=644']
  config.vm.synced_folder "repositories", "/home/vagrant/repositories",
    :create => 'true', :owner => 'vagrant', :group => 'vagrant',
    :mount_options => ['dmode=755', 'fmode=644']

  config.vm.provider "virtualbox" do |vb|
    vb.memory = "1024"
    vb.cpus = "2"
  end

  config.vm.boot_timeout = 3600

  # set auto_update to false, if you do NOT want to check the correct
  # additions version when booting this machine
  # config.vbguest.auto_update = false

  # Base provisioning
  config.vm.provision "shell", inline: <<-SHELL
    sudo yum -y update kernel
    sudo yum -y install kernel-devel kernel-headers dkms gcc gcc-c++
  SHELL
  config.vm.provision "reload"

  # ShangliLa constructing
  config.vm.provision "ansible_local" do |ansible|
    ansible.playbook = "/home/vagrant/ansible/local.yml"
    ansible.provisioning_path = "/home/vagrant/ansible"
  end

end