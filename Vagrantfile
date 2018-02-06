# -*- mode: ruby -*-
# vi: set ft=ruby :

require 'yaml'
require 'mkmf'
require 'fileutils'

# --------------------------------------------------------------------------------
# Define some variables
# --------------------------------------------------------------------------------
DIR = File.dirname(__FILE__)
config_file = File.join(DIR, 'config.yml')
site_config = YAML.load_file(config_file)

# --------------------------------------------------------------------------------
# Prevent logs from mkmf
# --------------------------------------------------------------------------------
module MakeMakefile::Logging
  @logfile = File::NULL
end

# --------------------------------------------------------------------------------
# Minimal Vagrant version
# --------------------------------------------------------------------------------
Vagrant.require_version '>= 1.5.1'
Vagrant.configure("2") do |config|

  config.vm.box = "bento/ubuntu-16.04"

  config.vm.network "private_network", ip: site_config['ip_addr']
  config.vm.hostname = site_config['hostname']

  config.ssh.insert_key = false

  config.ssh.forward_agent = true

  # Fix for: "stdin: is not a tty"
  # https://github.com/mitchellh/vagrant/issues/1673#issuecomment-28288042
  config.ssh.shell = %{bash -c 'BASH_ENV=/etc/profile exec bash'}

  config.vm.synced_folder "./sites", "/sites", :mount_options => ["dmode=777", "fmode=666"]

  config.vm.provision "shell" do |shell|
    shell.inline = "sudo apt-get update -y && DEBIAN_FRONTEND=noninteractive apt-get upgrade -yq && sudo apt-get install -y python"
  end

  config.vm.provision "ansible" do |ansible|
    ansible.playbook = "provision/main.yml"
  end

  # --------------------------------------------------------------------------------
  # Update hosts-file or prompt if hostsupdater-plugin is not installed && exit
  # --------------------------------------------------------------------------------
  if Vagrant.has_plugin? 'vagrant-hostsupdater'
    domains = get_domains(site_config)
    config.hostsupdater.aliases = domains
  else
    error "vagrant-hostsupdater missing, please install the plugin: vagrant plugin install vagrant-hostsupdater"
  end

  # --------------------------------------------------------------------------------
  # VBox-configuration
  # --------------------------------------------------------------------------------
  config.vm.provider 'virtualbox' do |vb|

    # Give VM access to all cpu cores on the host
    cpus = case RbConfig::CONFIG['host_os']
      when /darwin/ then `sysctl -n hw.ncpu`.to_i
      when /linux/ then `nproc`.to_i
      else 2
    end

    # Customize memory in MB
    vb.customize ['modifyvm', :id, '--memory', 4096]
    vb.customize ['modifyvm', :id, '--cpus', cpus]

    # Fix for slow external network connections
    vb.customize ['modifyvm', :id, '--natdnshostresolver1', 'on']
    vb.customize ['modifyvm', :id, '--natdnsproxy1', 'on']
  end

end

# --------------------------------------------------------------------------------
# Get domains
# --------------------------------------------------------------------------------

def get_domains(config)
  sites = []
  unless config['domains'].nil?
    config['domains'].each do |domain|
      sites << domain['name']

      unless domain['subdomains'].nil?
        domain['subdomains'].each do |sub, props|
          sites << "#{sub}.#{domain}"
        end
      end
    end
  end

  sites.uniq
end
