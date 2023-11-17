# apacheds_k3s_vagrant_libvirt

This Vagrant setup creates a VM, installs the [k3s Kubernetes
distribution](https://k3s.io/) and installs a
[ApacheDS](https://directory.apache.org/apacheds/) LDAP server inside the
Kubernetes cluster.

Default OS is openSUSE Leap 15.5. Although that can be changed in the
Vagrantfile, please beware that this will break the Ansible provisioning.

## Vagrant

1. You need vagrant obviously. And ansible. And git...
1. Fetch the box, per default this is `opensuse/Leap-15.5.x86_64`, using
   `vagrant box add opensuse/Leap-15.5.x86_64`.
1. Make sure the git submodules are fully working by issuing `git submodule init
   && git submodule update`
1. Run `vagrant up`
1. Check the state of the provisioning by using `kubectl` with the kubeconfig
   file you can find in `./ansible/k3s-kubeconfig`
   ```
   $ kubectl --kubeconfig ./ansible/k3s-kubeconfig get pods -n apacheds
   ```
1. Once the pod is running (the `READY` columns shows `1/1`), you can connect to
   the LDAP server using the VM's IP address. Use `vagrant addres k3s-host` to
   find the IP.
   Then run something like this (using `b1s` as the password):
   ```
   $ ldapsearch -H ldap://IP-ADDRESS-GOES-HERE -W -b dc=apacheds,dc=ojkastl,dc=de -D uid=admin,ou=system cn=hgranger
   Enter LDAP Password:
   # extended LDIF
   #
   # LDAPv3
   # base <dc=apacheds,dc=ojkastl,dc=de> with scope subtree
   # filter: cn=hgranger
   # requesting: ALL
   #

   # hgranger, Users, apacheds.ojkastl.de
   dn: CN=hgranger,OU=Users,DC=apacheds,DC=ojkastl,DC=de
   mail: hgranger@hogwarts.invalid
   telephoneNumber: 123-456-7890
   givenName: Hermione
   samAccountName: hgranger
   sn: Granger
   cn: hgranger
   objectClass: top
   objectClass: user
   title: Tester
   userPassword:: e1NTSEF9YVV5QWtwUEkyMTEzSklDZXB3dTBvRVV6MzVOZlU0eDhxMGpaQVE9PQ=
    =

    # search result
    search: 2
    result: 0 Success

    # numResponses: 2
    # numEntries: 1
   ```
1. Party!

## Cleaning up

Normally a `vagrant destroy` cleans up the virtual machine. In this setup this
is not enough, as the kubeconfig file will not be deleted.

Clean it up manually (`rm ansible/k3s-kubeconfig`) or by running the Ansible
playbook `playbook-DELETE_everything_to_start_from_scratch.yml`.

## Disabling the Ansible provisioning

In case you do not want Ansible to install teleport (because you want to install
it yourself), just comment out the following lines in the `Vagrantfile`:

```hcl
    node.vm.provision "ansible" do |ansible|
      ansible.playbook = "ansible/playbook-vagrant.yml"
    end # node.vm.provision
```

You also find all of the playbooks in the `ansible` folder.

## License

BSD-3-Clause

## Author Information

I am Johannes Kastl, reachable via git@johannes-kastl.de.
