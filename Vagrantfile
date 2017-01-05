# One Vagrantfile to rule them all!
#
# This is a generic Vagrantfile that can be used without modification in
# a variety of situations. Hosts and their properties are specified in
# `vagrant-hosts.yml`.
#
# See https://github.com/bertvv/ansible-skeleton/ for details
require 'rbconfig'
require 'yaml'

# Set your default base box here
DEFAULT_BASE_BOX = 'bertvv/centos72'

VAGRANTFILE_API_VERSION = '2'
PROJECT_NAME = '/' + File.basename(Dir.getwd)

# When set to `true`, Ansible will be forced to be run locally on the VM
# instead of from the host machine (provided Ansible is installed).
FORCE_LOCAL_RUN = false

hosts = YAML.load_file('vagrant-hosts.yml')

# {{{ Helper functions

def provision_ansible(config)
  if run_locally?
    # Provisioning configuration for shell script.
    config.vm.provision "ansible_local" do |ansible|
      ansible.playbook = 'ansible/site.yml'
      ansible.sudo = true
      ansible.install = true
      ansible.install_mode = ':pip'
      ansible.version = 'latest'
    end
  else
    # Provisioning configuration for Ansible (for Mac/Linux hosts).
    config.vm.provision 'ansible' do |ansible|
      ansible.playbook = 'ansible/site.yml'
      ansible.sudo = true
    end
  end
end

def run_locally?
  windows_host? || FORCE_LOCAL_RUN
end

def windows_host?
  RbConfig::CONFIG['host_os'] =~ /mswin|mingw|cygwin/
end

# Set options for the network interface configuration. All values are
# optional, and can include:
# - ip (default = DHCP)
# - netmask (default value = 255.255.255.0
# - mac
# - auto_config (if false, Vagrant will not configure this network interface
# - intnet (if true, an internal network adapter will be created instead of a
#   host-only adapter)
def network_options(host)
  options = {}

  if host.key?('ip')
    options[:ip] = host['ip']
    options[:netmask] = host['netmask'] ||= '255.255.255.0'
  else
    options[:type] = 'dhcp'
  end

  options[:mac] = host['mac'].gsub(/[-:]/, '') if host.key?('mac')
  options[:auto_config] = host['auto_config'] if host.key?('auto_config')
  options[:virtualbox__intnet] = true if host.key?('intnet') && host['intnet']
  options
end

def custom_synced_folders(vm, host)
  return unless host.key?('synced_folders')
  folders = host['synced_folders']
  folders.each do |folder|
    vm.synced_folder folder['src'], folder['dest'], folder['options']
  end
end

def custom_disks(vb,host)
  return unless host.key?('disks')
  disks = host['disks']
  disksDir = File.join(Dir.pwd, "disks")
  disks.each_with_index do  |disk,index|
    diskPort = index + 2
    diskFile = File.join(disksDir,host['name'],diskPort.to_s+".vdi")
    unless File.exist?('')
      vb.customize ['createhd', '--filename', diskFile, '--variant', 'Fixed', '--size', disk.size]
    end
    vb.customize ['storageattach', :id,  '--storagectl', 'SATA Controller', '--port', diskPort, '--device', 0, '--type', 'hdd', '--medium', diskFile]
  end
end
# }}}

Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|
  config.ssh.insert_key = false
  hosts.each do |host|
    config.vm.define host['name'] do |node|
      node.vm.box = host['box'] ||= DEFAULT_BASE_BOX
      node.vm.box_url = host['box_url'] if host.key? 'box_url'

      node.vm.hostname = host['name']
      node.vm.network :private_network, network_options(host)
      custom_synced_folders(node.vm, host)

      config.persistent_storage.enabled = true
      config.persistent_storage.location = "./test.vdi"
      config.persistent_storage.size = 5000

      node.vm.provider :virtualbox do |vb|
        # WARNING: if the name of the current directory is the same as the
        # host name, this will fail.
        vb.customize ['modifyvm', :id, '--groups', PROJECT_NAME]
       custom_disks(vb,host)
      end
    end
  end
  provision_ansible(config)
end

# -*- mode: ruby -*-
# vi: ft=ruby :
