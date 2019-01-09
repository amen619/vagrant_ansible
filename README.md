# Vagrant Ansible

Creates a local Vagrant environment to develop Ansible on your own local machine.

## Vagrantfile compatibility versions
- vagrant: 2.2.2
- virtualbox: 6.0.0 r127566

### Plugins

- vagrant cachier (https://github.com/fgrehm/vagrant-cachier)
- vagrant disksize (https://github.com/sprotheroe/vagrant-disksize)
- vagrant-hosts (https://github.com/oscar-stack/vagrant-hosts)
- vagrant-vbguest (https://github.com/dotless-de/vagrant-vbguest)

### Keys
Before starting, "ssh-keys" are required under "keys" folder. Private key will be placed in ansible-master host and "Public key" will be placed on the "slave hosts".

