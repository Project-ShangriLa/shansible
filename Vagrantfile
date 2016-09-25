# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure(2) do |config|

  config.vm.box = "centos/7"

  config.vm.hostname = "dev-shangrila"

  if Vagrant::Util::Platform.windows?
    ENV["VAGRANT_DETECTED_OS"] = ENV["VAGRANT_DETECTED_OS"].to_s + " cygwin"
  end

  # Disable automatic box update checking. If you disable this, then
  # boxes will only be checked for updates when the user runs
  # `vagrant box outdated`. This is not recommended.
  config.vm.box_check_update = false

  # Create a forwarded port mapping which allows access to a specific port
  # within the machine from a port on the host machine. In the example below,
  # accessing "localhost:8080" will access port 80 on the guest machine.
  # config.vm.network "forwarded_port", guest: 80, host: 8080

  config.vm.network "private_network", ip: "192.168.33.10"

  # If you are using Windows, it may be more faster and to use the RSync.
  # type: "rsync"
  config.vm.synced_folder "ansible", "/home/vagrant/ansible",
    :owner => 'vagrant', :group => 'vagrant',
    :mount_options => ['dmode=755', 'fmode=744']
  config.vm.synced_folder "repositories", "/home/vagrant/repositories",
    :create => 'true', :owner => 'vagrant', :group => 'vagrant',
    :mount_options => ['dmode=755', 'fmode=744']

  config.vm.provider "virtualbox" do |vb|
    vb.memory = "1024"
    vb.cpus = "2"
  end

  config.vm.boot_timeout = 5400

  config.ssh.forward_agent = true

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
    ansible.playbook = "/home/vagrant/ansible/site.yml"
    ansible.provisioning_path = "/home/vagrant/ansible"
    ansible.limit = "all"
    ansible.install = true
    ansible.inventory_path = "/home/vagrant/ansible/local"
  end

end
