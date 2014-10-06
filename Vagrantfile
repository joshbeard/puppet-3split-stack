# -*- mode: ruby -*-
# vi: set ft=ruby :

# Vagrantfile API/syntax version. Don't touch unless you know what you're doing!
VAGRANTFILE_API_VERSION = "2"

Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|
  # Every Vagrant virtual environment requires a box to build off of.
  config.vm.box = "puppetlabs/centos-6.5-64-nocm"

  instances = {

    :puppetmaster1 => {
      :ip   => '192.168.29.10',
      :mem  => '1024',
      :cpus => '2',
    },
    :puppetdb1 => {
      :ip   => '192.168.29.11',
      :mem  => '1024',
      :cpus => '2',
    },
    :puppetconsole1 => {
      :ip   => '192.168.29.12',
      :mem  => '512',
      :cpus => '2',
    },
    :puppetmaster2 => {
      :ip   => '192.168.29.20',
      :mem  => '512',
      :cpus => '1',
    },

    ## An agent
    :puppetagent1 => {
      :ip   => '192.168.29.30',
      :mem  => '512',
      :cpus => '1',
    },

  }

  instances.each do |vname, params|
    ## Build each system from above
    config.vm.define vname do |node|
      ## Configure Virtualbox
      config.vm.provider :virtualbox do |vb|
        # vb.customize ["startvm", :id, "--type", "gui"]
        vb.customize ["modifyvm", :id, "--memory", params[:mem]]
        vb.customize ["modifyvm", :id, "--cpus", params[:cpus]]

        ## This significantly improves performance on multicore VMs
        ## Yay learning the hard way
        if params[:cpus].to_i > 1
          vb.customize ["modifyvm", :id, "--ioapic", "on"]
        end
        ## Additional NICs
        #vb.customize ["modifyvm", :id, "--nictype1", "virtio"]
        #vb.customize ["modifyvm", :id, "--nictype2", "virtio"]
      end

      node.vm.hostname = "#{vname}.vagrant.vm"
      config.vm.network "private_network", ip: params[:ip]

      config.vm.synced_folder ".", "/vagrant", id: "vagrant-root"

      node.vm.provision :shell do |shell|
        shell.path = ".vagrant_provision.sh"
        shell.args = "#{vname}"
      end
    end
  end
end
