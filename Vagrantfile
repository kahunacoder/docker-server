# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure("2") do |config|
	config.vm.define "docker-vm"
  config.vm.hostname = "docker-vm"
  config.vm.box = "ubuntu/focal64" #Ubuntu 20.04 LTS 64bit
  config.vm.network :private_network, :auto_network => true
  config.vm.network "forwarded_port", guest: 9001, host: 9001
  config.vm.network "forwarded_port", guest: 443, host: 443
  config.vm.network "forwarded_port", guest: 80, host: 80
  config.ssh.forward_agent = true
  config.vm.provision :ansible do |ansible|
    # Uncomment below to see Ansible output. Add or subtract v's to see more or less.
    # ansible.verbose = "vvv"
    ansible.playbook = "playbook.yml"
  end
end
