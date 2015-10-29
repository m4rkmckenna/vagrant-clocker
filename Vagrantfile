# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.require_version ">= 1.6.0"
VAGRANTFILE_API_VERSION = 2
DEFAULT_BOX = "base"
DEFAULT_RAM = 256
DEFAULT_CPUS = 1

require 'yaml'

clockerConfig = YAML.load_file('clockerConfig.yaml')

Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|
  
  clockerConfig['master']['node'].each do |master_node|
    
    #Create MASTERs
    config.vm.define master_node['name'] do |master_node_config|
      master_node_config.vm.hostname = master_node['name']
      #Set BOX 
      if master_node.has_key?('box') then
        master_node_config.vm.box = master_node['box']
      elsif clockerConfig['master'].has_key?('box') then
        master_node_config.vm.box = clockerConfig['master']['box']
      else
         master_node_config.vm.box = DEFAULT_BOX
      end
      
      #Configure Network
      if master_node.has_key?("ip") then
        master_node_config.vm.network "private_network", ip: master_node["ip"]
      end
      
      #Configure Port forwards
      if master_node.has_key?("forwarded_ports") then
        master_node["forwarded_ports"].each do |ports|
          master_node_config.vm.network "forwarded_port", guest: ports["guest"], host: ports["host"], guest_ip: ports["guest_ip"]
        end
      end
      
      #Custom SHELL Provisioning
      if master_node.has_key?("shell")
        master_node["shell"].each do |cmd|
          master_node_config.vm.provision "shell", privileged: false, inline: cmd
        end
      end
      
      #Configure VIRTUALBOX
      master_node_config.vm.provider :virtualbox do |vb|
        vb.name = master_node["name"]
        #Configure RAM
        if master_node.has_key?('ram') then
          vb.memory = master_node['ram']
        elsif clockerConfig['master'].has_key?('ram') then
          vb.memory = clockerConfig['master']['ram']
        else
          vb.memory = DEFAULT_RAM
        end
        
        #Configure CPUs
        if master_node.has_key?('cpus') then
          vb.cpus = master_node['cpus']
        elsif clockerConfig['master'].has_key?('cpus') then
          vb.cpus = clockerConfig['master']['cpus']
        else
          vb.cpus = DEFAULT_CPUS
        end
      end
      
    end
  end
  
  #Create SLAVES
  (1..clockerConfig['slave']['count']).each do |i|
    config.vm.define vm_name = clockerConfig['slave']['hostname_template'] % [i] do |slave_node_config|
      slave_node_config.vm.hostname = vm_name
      #Set BOX
      if clockerConfig['slave'].has_key?('box') then
        slave_node_config.vm.box = clockerConfig['slave']['box']
      else
         master_node_config.vm.box = DEFAULT_BOX
      end
      slave_node_config.vm.network "private_network", ip: clockerConfig['slave']['ip_template'] % [i]
      #Custom SHELL Provisioning
      if clockerConfig['slave'].has_key?("shell")
        clockerConfig['slave']["shell"].each do |cmd|
          slave_node_config.vm.provision "shell", privileged: false, inline: cmd
        end
      end
      
      #Configure VIRTUALBOX
      slave_node_config.vm.provider :virtualbox do |vb|
        vb.name = vm_name
        #Configure RAM
        if clockerConfig['slave'].has_key?('ram') then
          vb.memory = clockerConfig['slave']['ram']
        else
          vb.memory = DEFAULT_RAM
        end
        
        #Configure CPUs
        if clockerConfig['slave'].has_key?('cpus') then
          vb.cpus = clockerConfig['slave']['cpus']
        else
          vb.cpus = DEFAULT_CPUS
        end
      end
    end 
  end
end
