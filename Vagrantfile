# -*- mode: ruby -*-
# vi: set ft=ruby :
require "yaml"

hosts_file = YAML.load_file("vagrant/hosts.yml")

vagrant_conf, ansible_master, nodes = hosts_file["vagrant_conf"], hosts_file["ansible_master"], hosts_file["nodes"]

Vagrant.configure("2") do |config|

    # mejoramos el rendimiento con el plugin vagrant-cachier
    if vagrant_conf["cachier"] and Vagrant.has_plugin?("vagrant-cachier")
        # con varias maquinas virtuales scope = :box da problemas
        #config.cache.scope = :box
        config.cache.scope = :machine
        config.cache.synced_folder_opts = {
            owner: "_apt",
            group: "vagrant",
            # si no ponemos estas opciones da error de permisos en /tmp/vagrant-cache
            mount_options: ["dmode=777", "fmode=666"]
        }
    end

    # ansible-master
    config.vm.define ansible_master["hostname"] do |ansiblemasterconfig|
        ansiblemasterconfig.vm.box = ansible_master["box"]
        ansiblemasterconfig.vm.box_url = ansible_master["box_url"]
        ansiblemasterconfig.vm.hostname = ansible_master["hostname"]
        ansiblemasterconfig.vm.network :private_network, ip: ansible_master["ip"]
        if Vagrant.has_plugin?("vagrant-hosts")
            ansiblemasterconfig.vm.provision :hosts, :sync_hosts => true
        end
        memory = ansible_master["ram"] ? ansible_master["ram"] : 1024;
        ansiblemasterconfig.vm.provider :virtualbox do |vb|
            vb.linked_clone = true
            vb.cpus = ansible_master["cpus"]
            vb.memory = memory
        end
        ansiblemasterconfig.vm.provision "ansible_local" do |ansible|
            ansible.playbook = "provisioning/playbook.yml"
        end
        # utilizamos rsync para que no se modifiquen los ficheros del host
        ansiblemasterconfig.vm.synced_folder vagrant_conf["synced_folders"]["provisioning"], "/home/vagrant/ansible", type: "rsync", rsync__exclude: vagrant_conf["rsync__exclude"], rsync__auto: vagrant_conf["rsync__auto"]
        ansiblemasterconfig.vm.provision "file", source: "keys/id_rsa", destination: "/tmp/id_rsa"          
        ansiblemasterconfig.vm.provision "shell", privileged: true, inline: "mv -f /tmp/id_rsa /home/vagrant/.ssh/id_rsa"
        # hostname a ipv6ls
        ansiblemasterconfig.vm.provision "shell", privileged: true, inline: "sed -i '/^::1/ s/$/ %s/' /etc/hosts" % ansible_master["hostname"]
        ansiblemasterconfig.vm.provision "shell", privileged: true, inline: "chmod 744 /home/vagrant/ansible; chmod 400 /home/vagrant/.ssh/id_rsa"
    end

    # nodes
    nodes.each do |node|
        config.vm.define node["hostname"] do |nodeconfig|
            nodeconfig.vm.box = node["box"]
            nodeconfig.vm.box_url = node["box_url"]
            nodeconfig.vm.hostname = node["hostname"]
            nodeconfig.disksize.size = node["disksize"]
            nodeconfig.vm.network :private_network, ip: node["ip"]
            if Vagrant.has_plugin?("vagrant-hosts")
                nodeconfig.vm.provision :hosts, :sync_hosts => true
            end
            memory = node["ram"] ? node["ram"] : 1024;
            nodeconfig.vm.provider :virtualbox do |vb|
                vb.linked_clone = true
                vb.cpus = node["cpus"]
                vb.memory = memory
                # vagrant 2.2.1 issue. Windows hosts winrm was not working
                # https://github.com/hashicorp/vagrant/issues/10429
                vb.default_nic_type = nil
                node["extra_disks"].each_with_index do |disk_size, disk_index|
                    disk_file = "vagrant/disks/%s_%d.vdi" % [node["hostname"], disk_index]
                    if not File.exist?(disk_file)
                        #vb.customize ['storagectl', :id, '--name', 'SATA Controller', '--add', 'sata', '--portcount', 4]
                        #vb.customize ['storageattach', :id,  '--storagectl', 'SATA Controller', '--port', 1, '--device', 0, '--type', 'hdd', '--medium', disk_file]
                        vb.customize ['createhd', '--filename', disk_file, '--variant', 'Fixed', '--size', disk_size]
                        vb.customize ['storageattach', :id,  '--storagectl', 'SCSI', '--port', 3 + disk_index, '--device', 0, '--type', 'hdd', '--medium', disk_file]
                    end
                end
            end
            # configuracion especifica de windows
            if node["box"].downcase.include? "windows"
                nodeconfig.vm.guest = :windows
                nodeconfig.vm.communicator = "winrm"
                nodeconfig.vm.boot_timeout = 600
                nodeconfig.vm.graceful_halt_timeout = 600
                nodeconfig.winrm.transport = :plaintext
                nodeconfig.winrm.username = node["user"]
                nodeconfig.winrm.password = node["password"]
            end
            # solo en maquinas linux configuramos los grains en un fichero
            if node["box"].downcase.include? "windows"
                nodeconfig.vm.provision "shell", inline: "New-Item -Path 'C:/' -Name 'testfile1.txt' -ItemType 'file' -Value 'This is a text string.'"
            else
                nodeconfig.vm.provision "ansible_local" do |ansible|
                    ansible.playbook = "provisioning/playbook.yml"
                end
                nodeconfig.vm.provision "file", source: "keys/id_rsa.pub", destination: "/tmp/id_rsa.pub"          
                nodeconfig.vm.provision "shell", privileged: true, inline: "cat /tmp/id_rsa.pub >> /home/vagrant/.ssh/authorized_keys"
            end
            
        end
    end

end