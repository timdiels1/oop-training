# -*- mode: ruby -*-
# vi: set ft=ruby :

## Check dependencies

# vagrant version
Vagrant.require_version ">= 1.7.0"


# vagrant-triggers plugin
if !Vagrant.has_plugin?("vagrant-triggers")
  puts "The 'vagrant-triggers' plugin is required, install it with the following command:"
  puts " vagrant plugin install vagrant-triggers"
  exit
end
# vagrant-bindfs plugin
if !Vagrant.has_plugin?("vagrant-bindfs")
  puts "The 'vagrant-bindfs' plugin is required, install it with the following command:"
  puts " vagrant plugin install vagrant-bindfs"
  exit
end


Vagrant.configure("2") do |config|

  # General configuration, valid for all defined boxes.

  # Virtualbox tweaks.
  config.vm.provider :virtualbox do |vb|
   # Moar memory
    vb.customize ["modifyvm", :id, "--memory", "2000"]
    # allow symlinks
    vb.customize ["setextradata", :id, "VBoxInternal2/SharedFoldersEnableSymlinksCreate/vagrant", "1"]
  end

  # NFS shares
  config.vm.synced_folder "www", "/var/www", :nfs => true
  config.vm.synced_folder "db", "/home/vagrant/db", :nfs => true
  config.vm.synced_folder "scripts", "/home/vagrant/scripts", :nfs => { :mount_options => ["mfsymlinks"] }

  #config.vm.synced_folder "www", "/home/vagrant/www", type: "nfs"
  #config.bindfs.bind_folder "/home/vagrant/www", "/var/www", owner: "vagrant", group: "vagrant", perms: "0777"

  # https/ssl port forwarding
  config.vm.network :forwarded_port, guest: 443, host: 4444

  # SSH
  config.ssh.shell = "BASH_ENV=/etc/profile bash -l"

  ## Provision
  # setup.sh will run to provide additional provisioning, see script for details
  config.vm.provision :shell, :inline => "
    /vagrant/scripts/setup.sh;
  "

  ## Triggers

  # cleanup actions that will run on 'destroy'
  config.trigger.before :destroy do
    for script in Dir.glob("scripts/cleanup/*")
      begin
        answer = ''
        info("Do you want to execute the '#{script}' cleanup script'? [Y/n] ")
        Timeout::timeout(5) { answer = $stdin.gets }
      rescue Timeout::Error
        answer = 'y'
      end
      if answer == 'Y' or answer == 'y'
        run "vagrant ssh -c 'sh #{script}'"
      end
    end
  end

  # the box
  config.vm.define :vagrant do |vagrant|
    vagrant.vm.network :private_network, ip: "192.168.50.2"
    vagrant.vm.hostname = "vagrant.loc"
  end
end

if File.file?('Vagrantfile.box')
  load 'Vagrantfile.box'
end
