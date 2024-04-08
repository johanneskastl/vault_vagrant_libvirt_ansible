Vagrant.configure("2") do |config|

  ###################################################################################
  config.vm.define "vault-client" do |node|

    # which image to use
    node.vm.box = "opensuse/Leap-15.5.x86_64"

    # sizing of the VMs
    node.vm.provider "libvirt" do |lv|
      lv.random_hostname = true
      lv.memory = 1024
      lv.cpus = 2
    end

    # set the hostname
    node.vm.hostname = "vault-client"

    # disable shared folders
    node.vm.synced_folder ".", "/vagrant", disabled: true

  end # config.vm.define agents

  ###################################################################################
  config.vm.define "vault" do |node|

    # which image to use
    node.vm.box = "opensuse/Leap-15.5.x86_64"

    # sizing of the VMs
    node.vm.provider "libvirt" do |lv|
      lv.random_hostname = false
      lv.memory = 8196
      lv.cpus = 2
    end

    # set the hostname
    node.vm.hostname = "vault"

    # disable shared folders
    node.vm.synced_folder ".", "/vagrant", disabled: true

    # Ansible
    node.vm.provision "ansible" do |ansible|
      ansible.compatibility_mode = "2.0"
          ansible.limit = "all"
          ansible.groups = {
            "vault_clients"  => [ "vault-client" ],
            "vault_server"  => [ "vault" ]
          }
      ansible.playbook = "ansible/playbook-vagrant.yml"
    end # node.vm.provision

    node.trigger.after :destroy do |trigger|
      trigger.warn = "Removing ansible/vault_root_token"
      trigger.run = {inline: "rm -vf ansible/vault_root_token"}
    end

  end # config.vm.define

end # Vagrant.configure
