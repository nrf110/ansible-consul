# -*- mode: ruby -*-
# vi: set ft=ruby :
require 'yaml'

cluster_config = YAML::load_file(File.join(__dir__, 'cluster.yaml'))

Vagrant.configure("2") do |config|
  config.vm.box = "bento/ubuntu-16.04"
  config.vm.network "private_network", type: "dhcp"

  config.hostmanager.enabled = true
  config.hostmanager.manage_host = true
  config.hostmanager.include_offline = true
  config.hostmanager.ignore_private_ip = false

  cached_addresses = {}
  config.hostmanager.ip_resolver = proc do |vm, resolving_vm|
    if cached_addresses[vm.name].nil?
      if hostname = (vm.ssh_info && vm.ssh_info[:host])
        vm.communicate.execute("hostname -I | cut -d ' ' -f 2") do |type, contents|
          cached_addresses[vm.name] = contents.split("\n").first[/(\d+\.\d+\.\d+\.\d+)/, 1]
        end
      end
    end
    cached_addresses[vm.name]
  end

  groups = {}
  cluster_config.each do |group_name, count|
    groups[group_name] = (1..count).map { |idx| "#{group_name.gsub(/_/, '.')}#{idx}" }
  end

  machines = groups.values.reduce([]) { |a, b| a | b }
  machines.each_with_index do |name,idx|
    config.vm.define name do |instance|
      instance.vm.hostname = name

      instance.vm.provision "shell" do |s|
        s.inline = "sudo apt-get install -y python"
      end

      if name == machines.last
        instance.vm.provision "ansible" do |ansible|
          ansible.groups = groups
          ansible.limit = 'all'
          ansible.playbook = 'test.yml'
          ansible.verbose = 'vv'
          ansible.sudo = true
        end
      end
    end
  end
end
