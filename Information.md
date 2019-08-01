### Running a playbook

```sh
$> ansible-playbook <playbookname.yml>
```

### Gathering facts

- [Ansible documentation](https://docs.ansible.com/ansible/latest/user_guide/playbooks_variables.html)

In Vagrant we can't access facts from another hosts. To do so, we can store facts in a [json file](https://docs.ansible.com/ansible/latest/user_guide/playbooks_variables.html#fact-caching) also.

```sh
$> ansible -m setup linux
$> ansible -m setup apache2
$> ansible -m setup <hostname> -a "filter=<factname>"
$> ansible -m setup apache2 -a "filter=ansible_os_family"
$> ansible -m setup apache2 -a "filter=ansible_distribution_release"
```

#### Creating a skeleton role:
- [Ansible documentation](https://docs.ansible.com/ansible/latest/reference_appendices/galaxy.html#ansible-galaxy)
```sh
$> cd roles
$> ansible-galaxy init <rolename>
```

### Debugging
In the tasks file we add "debug task":

```sh
- debug: var=hostvars['loadbalancer']["ansible_enp0s8"]["ipv4"]["address"]
- debug: var=hostvars['loadbalancer']["ansible_facts"]["os_family"]
- debug: var=hostvars['loadbalancer']["ansible_facts"]["nodename"]
```

Then we run the playbook (with start at task we indicate in which task we want to start):
```sh
$> ansible-playbook loadbalancer.yml --start-at-task "copy custom web"
```

We should see something like this:
```sh
TASK [nginx : debug] ***********************************************************************************************************
ok: [loadbalancer] => {
    "hostvars['loadbalancer']['ansible_enp0s8']['ipv4']['address']": "10.0.1.11"
}

TASK [nginx : debug] ***********************************************************************************************************
ok: [loadbalancer] => {
    "hostvars['loadbalancer']['ansible_facts']['os_family']": "Debian"
}

TASK [nginx : debug] ***********************************************************************************************************
ok: [loadbalancer] => {
    "hostvars['loadbalancer']['ansible_facts']['nodename']": "loadbalancer"
}
```

### Other links

- [Using Variables - facts-caching](https://docs.ansible.com/ansible/latest/user_guide/playbooks_variables.html#fact-caching)
- [Multi-Machine Vagrant Ansible Gotcha](https://blog.wjlr.org.uk/2014/12/30/multi-machine-vagrant-ansible-gotcha.html)
