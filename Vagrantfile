# -*- mode: ruby -*-
# vi: set ft=ruby :


Vagrant.configure("2") do |config|
  config.vm.box = "centos/8"
  config.ssh.insert_key = false
  # vagrant plugin install vagrant-vbguest
  # https://github.com/dotless-de/vagrant-vbguest
  if Vagrant.has_plugin?("vagrant-vbguest")
    config.vbguest.auto_update = false
    config.vbguest.no_remote = true
  end

  N = 1
  (1..N).each do |node_id|
    config.vm.define "node#{node_id}" do |node|
      node.vm.provider "virtualbox" do |vb|
        vb.memory = 1200
        vb.cpus = 2
      end
      node.vm.network "private_network", ip: "172.17.193.#{20+node_id}"
      node.vm.network "forwarded_port", guest: 9090, host:"#{9090+node_id}" 
      if node_id ==1
        node.vm.host_name = "app1"
      end
    end
  end

  config.vm.define "master" do |master|
    master.vm.provider "virtualbox" do |vb|
      vb.memory = 2500
      vb.cpus = 3
    end
    master.vm.host_name = "master"
    master.vm.network "private_network", ip: "172.17.193.20"

    sharedPath = '/vagrant'
    master.vm.synced_folder '.', sharedPath #, disabled: true
    master.vm.network "forwarded_port", guest: 9090, host:9090
    master.vm.provision "shell", inline: <<-SHELL
      dnf -y install python3-pip
      su vagrant -c 'pip3 install ansible --user'
    SHELL
    # vagrant plugin install vagrant-guest_ansible
    # https://github.com/vovimayhem/vagrant-guest_ansible
    provisioner = Vagrant::Util::Platform.windows? ? :guest_ansible : :ansible
    master.vm.provision provisioner do |ansible|
      ansible.inventory_path = "ansible_inventory"
      ansible.playbook = "playbooks/ansible-setup.yaml"
    end
    master.vm.provision provisioner do |ansible|
      ansible.inventory_path = "ansible_inventory"
      ansible.playbook = "playbooks/ansible-cockpit.yaml"
    end
  end
end
