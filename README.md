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
1. At the end of the Ansible provisioning, Ansible prints out a message like the
   following:

   ```bash
   TASK [Output URL] *******************************************************************************
   ok: [vault-client] => {
       "msg": "The website containing the 'secret' from Vault is reachable at http://192.0.2.13"
   }
   ```

1. Open the URL that Ansible displayed at the end of the run. You should see the
   secret value that Ansible wrote to Vault previously, fetched from Vault by
   the Vault agent and saved into Nginx's `index.html` file:

```bash
$ curl http://192.0.2.13
<!DOCTYPE html>
<html>
<body>

<h1>vault_vagrant_libvirt_ansible</h1>

<p>The secret stored in Vault is: vagrant-libvirt</p>


</body>
</html>
```

1. Log in on the vault-client VM using `vagrant ssh vault-client`.
1. Ansible already added a line to `.bashrc` for the `root` and `vagrant` users
   to set the Vault address:

   ```bash
   export VAULT_ADDR='http://192.0.2.13:8200'
   ```

1. You should already be logged in and can start working with Vault.
1. Play around with Vault, e.g. get information on your token using `vault token
   lookup`.
1. Have a look around the Vault server's WebUI, whose URL Ansible also prints
   out:

   ```bash
   TASK [Output Vault URL] ********************************************************
   ok: [vault-client] => {
       "msg": "The Vault Server is reachable at: http://192.168.2.13:8200"
   }
   ```

1. Party!

## Cleaning up

The VMs can be torn down after playing around using `vagrant destroy`. There is
one file that needs to be removed manually. This file contains the Vault root
token and is located at `ansible/vault_root_token`.

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
