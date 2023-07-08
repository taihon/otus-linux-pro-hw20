# -*- mode: ruby -*-
# vim: set ft=ruby :
home = ENV['HOME']
ENV["LC_ALL"] = "en_US.UTF-8"

MACHINES = {
  :router1 => {
        :box_name => "centos/7",
        :box_version => "2004.01",
        :net => [
          {ip: '10.0.10.1', adapter: 2, netmask: "255.255.255.252", virtualbox__intnet: "r1-r2"},
          {ip: '10.0.12.1', adapter: 3, netmask: "255.255.255.252", virtualbox__intnet: "r1-r3"},
          {ip: '192.168.10.1', adapter: 4, netmask: "255.255.255.0", virtualbox__intnet: "net1"},
          {ip: '192.168.11.101', adapter: 5},
       ]
  },
  :router2 => {
        :box_name => "centos/7",
        :box_version => "2004.01",
        :net => [
          {ip: '10.0.10.2', adapter: 2, netmask: "255.255.255.252", virtualbox__intnet: "r1-r2"},
          {ip: '10.0.11.2', adapter: 3, netmask: "255.255.255.252", virtualbox__intnet: "r2-r3"},
          {ip: '192.168.20.1', adapter: 4, netmask: "255.255.255.0", virtualbox__intnet: "net2"},
          {ip: '192.168.11.102', adapter: 5},
       ]
  },
  :router3 => {
        :box_name => "centos/7",
        :box_version => "2004.01",
        :net => [
          {ip: '10.0.11.1', adapter: 2, netmask: "255.255.255.252", virtualbox__intnet: "r2-r3"},
          {ip: '10.0.12.2', adapter: 3, netmask: "255.255.255.252", virtualbox__intnet: "r1-r3"},
          {ip: '192.168.30.1', adapter: 4, netmask: "255.255.255.0", virtualbox__intnet: "net3"},
          {ip: '192.168.11.103', adapter: 5},
       ]
  },
}

Vagrant.configure("2") do |config|

    config.vm.box_version = "2004.01"
    MACHINES.each_with_index do |(boxname, boxconfig), index|
        config.vm.synced_folder ".","/vagrant", disabled: true
        config.vm.define boxname do |box|
  
          box.vm.box = boxconfig[:box_name]
          box.vm.host_name = boxname.to_s
          box.vm.provider :virtualbox do |vb|
            vb.customize ["modifyvm", :id, "--memory", "192"]
            vb.customize ["modifyvm", :id, "--cpus", "1"]
          end
          boxconfig[:net].each do |ipconf|
            box.vm.network "private_network", ipconf
          end
          
          if boxconfig.key?(:public)
            box.vm.network "public_network", boxconfig[:public]
          end
  
          box.vm.provision "shell", inline: <<-SHELL
            mkdir -p ~root/.ssh
            cp ~vagrant/.ssh/auth* ~root/.ssh
            echo "net.ipv4.conf.all.forwarding=1" >> /etc/sysctl.d/netforward.conf
            echo "net.ipv4.conf.all.rp_filter=2" >> /etc/sysctl.d/netforward.conf
            sysctl --system > /dev/null
          SHELL
          
        if index == MACHINES.size - 1
           box.vm.provision "ansible" do |ansible|
            ansible.playbook = "ansible/playbook.yml"
            ansible.limit = "all"
            ansible.inventory_path = "ansible/hosts"
            ansible.host_key_checking = false
          end
        end
    	end
  	end
end