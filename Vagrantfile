ENV['VAGRANT_DEFAULT_PROVIDER'] = 'digital_ocean'

# We use YAML to load an external config file with a bunch of config that should not
# be committed to version control.
require 'yaml'
private = YAML::load_file("./puppet/hieradata/private.yaml")

Vagrant.configure("2") do |config|

  # Local specific vagrant config.
  config.vm.define "local", primary: true do |local| 
    local.vm.box = "precise64"
    local.vm.box_url = "http://files.vagrantup.com/precise64.box" 
    local.vm.network :private_network, ip: "192.168.33.11"
    local.ssh.forward_agent = true

    local.vm.provider :virtualbox do |v|
      v.customize ["modifyvm", :id, "--natdnshostresolver1", "on"]
      v.customize ["modifyvm", :id, "--memory", 4096]
    end
  end

  # Digital Ocean vagrant config
  config.vm.define "remote" do |remote|
    remote.vm.box = "digital_ocean"
    remote.vm.box_url = "https://github.com/smdahlen/vagrant-digitalocean/raw/master/box/digital_ocean.box"
    remote.ssh.private_key_path = "~/.ssh/id_rsa"
    remote.ssh.username = private['vagrantfile-digital_ocean']['ssh']['username']
    remote.ssh.shell = "bash -l"
    remote.ssh.keep_alive = true
    remote.ssh.forward_agent = false
    remote.ssh.forward_x11 = false
    remote.vagrant.host = :detect

    remote.vm.provider :digital_ocean do |provider|
      provider.client_id = private['vagrantfile-digital_ocean']['vm']['provider']['digital_ocean']['client_id']
      provider.api_key = private['vagrantfile-digital_ocean']['vm']['provider']['digital_ocean']['api_key']
      provider.image = "Ubuntu 12.04 x64"
      provider.region = "New York 1"
      provider.size = "4GB"
    end
  end
    

  # Global vagrant config.
  config.vm.hostname = 'kindred'
  config.vm.synced_folder "../htdocs", "/var/www", id: "htdocs", :owner => "www-data", :group => "www-data"

  config.vm.provision :shell, :path => "shell/initial-setup.sh"
  config.vm.provision :shell, :path => "shell/update-puppet.sh"
  config.vm.provision :shell, :path => "shell/librarian-puppet-vagrant.sh"
  
  config.vm.provision :puppet do |puppet|
    puppet.facter = {
      "ssh_username" => private['vagrantfile-digital_ocean']['ssh']['username'],
      "neo4j_username" => private['neo4j']['username'],
      "neo4j_password" => private['neo4j']['password']
    }

    puppet.manifests_path = "puppet/manifests"
    puppet.options = ["--verbose", "--hiera_config /vagrant/hiera.yaml", "--parser future"]
  end

end
