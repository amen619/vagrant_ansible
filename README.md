# Vagrant Ansible

Creates a local Vagrant environment to develop Ansible on your own local machine.

## Vagrantfile compatibility versions
- vagrant: 2.2.4
- virtualbox: 6.0.8 r130520 (Qt5.9.5)

## Vagrant required plugins

- vagrant-cachier (https://github.com/fgrehm/vagrant-cachier)
```sh
$ vagrant plugin install vagrant-cachier
```
- vagrant-disksize (https://github.com/sprotheroe/vagrant-disksize)
```sh
$ vagrant plugin install vagrant-disksize
```
- vagrant-hosts (https://github.com/oscar-stack/vagrant-hosts)
```sh
$ vagrant plugin install vagrant-hosts
```
- vagrant-vbguest (https://github.com/dotless-de/vagrant-vbguest) 
```sh
$ vagrant plugin install vagrant-vbguest
```

### Keys
Before starting, "ssh-keys" are required under "keys" folder. Private key will be placed in ansible-master host and "Public key" will be placed on the "slave hosts".

