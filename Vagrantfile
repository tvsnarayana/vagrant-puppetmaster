# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure("2") do |config|

###############################################################################
# Base box                                                                    #
###############################################################################
    config.vm.box = "puppetlabs/centos-6.6-64-puppet"

###############################################################################
# Global plugin settings                                                #
###############################################################################
    unless Vagrant.has_plugin?("vagrant-hostmanager")
      raise 'vagrant-hostmanager is not installed!'
    end

###############################################################################
# Global provisioning settings                                                #
###############################################################################
    default_env = 'production'
    ext_env = ENV['VAGRANT_PUPPET_ENV']
    env = ext_env ? ext_env : default_env
    PUPPET = "sudo puppet agent -t --environment #{env} --server puppet.foreman.vagrant; echo $?"

###############################################################################
# Global VirtualBox settings                                                  #
###############################################################################
    config.vm.provider 'virtualbox' do |v|
    v.customize [
      'modifyvm', :id,
      '--groups', '/Vagrant/puppetmaster'
    ]
    end

###############################################################################
# Global /etc/hosts file settings                                                  #
###############################################################################
    config.hostmanager.enabled = true
    config.hostmanager.manage_host = true
    config.hostmanager.ignore_private_ip = false
    config.hostmanager.include_offline = true

###############################################################################
# VM definitions                                                              #
###############################################################################

    config.vm.define :puppet do |puppet_config|
      puppet_config.vm.host_name = "puppet.foreman.vagrant"
      puppet_config.vm.network :private_network, ip: "192.168.21.130"
      puppet_config.vm.synced_folder 'manifests/', "/etc/puppet/environments/#{env}/manifests"
      puppet_config.vm.synced_folder 'modules/', "/etc/puppet/environments/#{env}/modules"
      puppet_config.vm.synced_folder 'hiera/', '/var/lib/hiera'
      puppet_config.vm.provision :shell, inline: 'sudo cp /vagrant/files/hiera.yaml /etc/puppet/hiera.yaml'
      puppet_config.vm.provision :shell, inline: 'sudo cp /vagrant/files/autosign.conf /etc/puppet/autosign.conf'
      puppet_config.vm.provision :puppet do |puppet|
          puppet.options = "--environment #{env}"
          puppet.manifests_path = "manifests"
          puppet.manifest_file  = ""
          puppet.module_path = "modules"
          puppet.hiera_config_path = "files/hiera.yaml"
      end
    end

    config.vm.define :db do |db_config|
      db_config.vm.host_name = "db.foreman.vagrant"
      db_config.vm.network :forwarded_port, guest: 22, host: 2131
      db_config.vm.network :private_network, ip: "192.168.21.131"
      db_config.vm.provision 'shell', inline: PUPPET
    end

    config.vm.define :puppetdb do |puppetdb_config|
      puppetdb_config.vm.host_name = "puppetdb.foreman.vagrant"
      puppetdb_config.vm.network :forwarded_port, guest: 22, host: 2132
      puppetdb_config.vm.network :private_network, ip: "192.168.21.132"
      puppetdb_config.vm.provision 'shell', inline: PUPPET
    end

    config.vm.define :foreman do |foreman_config|
      foreman_config.vm.host_name = "foreman.foreman.vagrant"
      foreman_config.vm.network :forwarded_port, guest: 22, host: 2133
      foreman_config.vm.network :private_network, ip: "192.168.21.133"
      foreman_config.vm.provision 'shell', inline: PUPPET
    end

    config.vm.define :node do |node_config|
      node_config.vm.host_name = "node.foreman.vagrant"
      node_config.vm.network :forwarded_port, guest: 22, host: 2140
      node_config.vm.network :private_network, ip: "192.168.21.140"
      node_config.vm.provision 'shell', inline: PUPPET
    end
end
