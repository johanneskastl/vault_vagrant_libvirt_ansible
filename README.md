# vault_vagrant_libvirt_ansible

This Vagrant setup creates a VM and configures [a Hashicorp Vault
server](https://www.hashicorp.com/products/vault). It also creates a VM and
installs the [Vault Agent
service](https://developer.hashicorp.com/vault/tutorials/vault-agent/agent-quick-start).

Default OS is openSUSE Leap 15.5. Although that can be changed in the
Vagrantfile, please beware that this will break the Ansible provisioning.

The server is only running in development mode. This means that the server does
not need to be unsealed after the start. This also means that all configuration
is only stored in-memory aka is lost after a restart or a reboot of the VM.
Enough for playing around, but **do NOT use this in production**

The Vault agent is using the root token to authenticate to the Vault server.
This is also something that needs to be done properly in a production setup, but
is enough for a demo.

## Vagrant

1. You need vagrant obviously. And ansible. And git...
1. Fetch the box, per default this is `opensuse/Leap-15.5.x86_64`, using
   `vagrant box add opensuse/Leap-15.5.x86_64`.
1. Make sure the git submodules are fully working by issuing `git submodule init
   && git submodule update`
1. Run `vagrant up`
1. Log in on the vault-client VM using `vagrant ssh vault-client`.
1. Get the token from `/var/lib/vault/vault-token-via-agent`
1. Run the following commands and replace `192.0.2.13` with the vault server
   VM's IP address:

   ```bash
   export VAULT_ADDR='http://192.0.2.13:8200'
   ```

1. Log in using the token you got from the
   `/var/lib/vault/vault-token-via-agent` file:

   ```bash
   $ vault login
   Token (will be hidden):
   $
   ```

1. Play around with Vault, e.g. get information on your token using `vault token
   lookup`.
1. Party!

## Disabling the Ansible provisioning

In case you do not want Ansible to install Vault (because you want to install
it yourself), just comment out the following lines in the `Vagrantfile`:

```hcl
    node.vm.provision "ansible" do |ansible|
      ansible.compatibility_mode = "2.0"
          ansible.limit = "all"
          ansible.groups = {
            "vault_clients"  => [ "vault-client" ],
            "vault_server"  => [ "vault" ]
          }
      ansible.playbook = "ansible/playbook-vagrant.yml"
    end # node.vm.provision
```

You also find all of the playbooks in the `ansible` folder.

## License

BSD-3-Clause

## Author Information

I am Johannes Kastl, reachable via git@johannes-kastl.de
