##### Ansible installation guide

https://docs.ansible.com/ansible/latest/installation_guide/index.html

https://docs.ansible.com/ansible/latest/installation_guide/intro_installation.html#installing-and-upgrading-ansible-with-pip

```
$ python3 -V
Python 3.9.18

$ sudo dnf install python3-pip

$ python3 -m pip -V
pip 21.2.3 from /usr/lib/python3.9/site-packages/pip (python 3.9)

$ pip -V
pip 21.2.3 from /usr/lib/python3.9/site-packages/pip (python 3.9)
```

##### Installing Ansible

Use `pip` in your selected Python environment to install the full Ansible package for the current user:

```
$ python3 -m pip install --user ansible
```

##### Upgrading Ansible

To upgrade an existing Ansible installation in this Python environment to the latest released version, simply add `--upgrade` to the command above:

```
$ python3 -m pip install --upgrade --user ansible
```

```
$ ansible --version
ansible [core 2.15.9]
  config file = None
  configured module search path = ['/home/hong/.ansible/plugins/modules', '/usr/share/ansible/plugins/modules']
  ansible python module location = /home/hong/.local/lib/python3.9/site-packages/ansible
  ansible collection location = /home/hong/.ansible/collections:/usr/share/ansible/collections
  executable location = /home/hong/.local/bin/ansible
  python version = 3.9.18 (main, Jan  4 2024, 00:00:00) [GCC 11.4.1 20230605 (Red Hat 11.4.1-2)] (/usr/bin/python3)
  jinja version = 3.1.3
  libyaml = True
```



---

Add `ansible-node1`, `ansible-node2` to `/etc/hosts`

```
192.168.0.39 ansible-controller
IP_ADDRESS_OR_FQDN_OF_NODE1 ansible-node1
IP_ADDRESS_OR_FQDN_OF_NODE2 ansible-node2
```

`ssh` into both nodes.

---



##### Building Ansible inventories

https://docs.ansible.com/ansible/latest/inventory_guide/index.html

```
mkdir -p ~/ansible/inventory
cd ~/ansible/inventory
touch ~/ansible/inventory/inventory.ini
```

`inventory.ini`:

```
ansible-node1 ansible_connection=ssh ansible_user=ubuntu ansible_ssh_private_key_file=~/.ssh/srv_id_ed25519
ansible-node2 ansible_connection=ssh ansible_user=ubuntu ansible_ssh_private_key_file=~/.ssh/srv_id_ed25519
```

```
$ # ansible ping all nodes
$ ansible all -m ping -i inventory.ini
ansible-node2 | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python3"
    },
    "changed": false,
    "ping": "pong"
}
ansible-node1 | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python3"
    },
    "changed": false,
    "ping": "pong"
}

$ # ansible ping one node
$ ansible ansible-node1 -m ping -i inventory.ini
ansible-node1 | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python3"
    },
    "changed": false,
    "ping": "pong"
}
```

The headings in brackets are group names. Dash ("-") is invalid.

https://docs.ansible.com/ansible/latest/playbook_guide/playbooks_variables.html#valid-variable-names

`inventory.ini` with groups:

```
[web_srv_1]
ansible-node1 ansible_connection=ssh ansible_user=ubuntu ansible_ssh_private_key_file=~/.ssh/srv_id_ed25519

[web_srv_2]
ansible-node2 ansible_connection=ssh ansible_user=ubuntu ansible_ssh_private_key_file=~/.ssh/srv_id_ed25519
```

```
$ ansible web_srv_1 -m ping -i inventory.ini
ansible-node1 | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python3"
    },
    "changed": false,
    "ping": "pong"
}
```

##### Adding ranges of hosts

If you have a lot of hosts with a similar pattern, you can add them as a range rather than listing each hostname separately:

```
[web_srv]
ansible-node[1:2] ansible_connection=ssh ansible_user=ubuntu ansible_ssh_private_key_file=~/.ssh/srv_id_ed25519

#ansible-node2 ansible_connection=ssh ansible_user=ubuntu ansible_ssh_private_key_file=~/.ssh/srv_id_ed25519
```



---

YAML guide

https://circleci.com/blog/what-is-yaml-a-beginner-s-guide/

---



##### Using Ansible playbooks

https://docs.ansible.com/ansible/latest/playbook_guide/index.html

```
mkdir -p ~/ansible/playbook && cd ~/ansible/playbook
touch ~/ansible/playbook/playbook1.yaml
```

Create `~/.ssh/config`

```
Host ansible-node1
  HostName                  IP_ADDRESS_OR_FQDN_OF_NODE1
  Port                      22
  User                      ubuntu
  IdentityFile              ~/.ssh/srv_id_ed25519
  ServerAliveInterval       5
  ExitOnForwardFailure      yes

Host ansible-node2
  HostName                  IP_ADDRESS_OR_FQDN_OF_NODE2
  Port                      22
  User                      ubuntu
  IdentityFile              ~/.ssh/srv_id_ed25519
  ServerAliveInterval       5
  ExitOnForwardFailure      yes
```

Create `inventory.ini` under `~/ansible/playbook/`:

```
[web_srv]
ansible-node1 ansible_connection=ssh
ansible-node2 ansible_connection=ssh
```

```
$ ls -l ~/ansible/playbook/
total 8
-rw-r--r--. 1 hong hong 84 Feb  4 22:29 inventory.ini
-rw-r--r--. 1 hong hong 72 Feb  4 22:21 playbook1.yaml
```

`playbook1.yaml`

```
- hosts: web_srv
  name: play-demo
  tasks:
  - name: verify host connectivity
    ping:
```

```
$ ansible-playbook playbook1.yaml -i inventory.ini --verbose

PLAY [play-demo] ************************************************************

TASK [Gathering Facts] ******************************************************
ok: [ansible-node1]
ok: [ansible-node2]

TASK [verify host connectivity] *****************************************************************************
ok: [ansible-node2] => {"changed": false, "ping": "pong"}
ok: [ansible-node1] => {"changed": false, "ping": "pong"}

PLAY RECAP ******************************************************************
ansible-node1              : ok=2    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
ansible-node2              : ok=2    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
```

##### Running playbooks in check mode

```
$ ansible-playbook --check playbook1.yaml -i inventory.ini
```



---



##### Using Ansible modules and plugins

https://docs.ansible.com/ansible/latest/module_plugin_guide/index.html

https://docs.ansible.com/ansible/2.9/modules/list_of_all_modules.html



You can execute modules from the command line.

```
$ ansible web_srv -m command -a "hostnamectl" -i inventory.ini
ansible-node1 | CHANGED | rc=0 >>
 Static hostname: ubuntu
       Icon name: computer-vm
         Chassis: vm
      Machine ID: d1c64eb552a0e0a30757820865c00979
         Boot ID: 5f9f51fec27e4507917cd78177b6ed3a
  Virtualization: vmware
Operating System: Ubuntu 22.04.3 LTS
          Kernel: Linux 6.5.0-15-generic
    Architecture: x86-64
 Hardware Vendor: VMware, Inc.
  Hardware Model: VMware Virtual Platform
ansible-node2 | CHANGED | rc=0 >>
 Static hostname: ubuntu
       Icon name: computer-vm
         Chassis: vm
      Machine ID: e8d6c2103cc5f604df24cdbf65c009b0
         Boot ID: df42dbcec69d431a827dbe2f05c527bc
  Virtualization: vmware
Operating System: Ubuntu 22.04.3 LTS
          Kernel: Linux 6.5.0-15-generic
    Architecture: x86-64
 Hardware Vendor: VMware, Inc.
  Hardware Model: VMware Virtual Platform
```

From playbooks, Ansible modules are executed in a very similar way. `playbook2.yaml`

```
- hosts: web_srv
  name: module-demo
  tasks:
  - name: query hostname and related settings
    command: hostnamectl
```

```
$ ansible-playbook playbook2.yaml -i inventory.ini --verbose
```

You can access the documentation for each module from the command line with the ansible-doc tool.

```
$ ansible-doc command
> ANSIBLE.BUILTIN.COMMAND    (/home/hong/.local/lib/python3.9/site-packages/ansible/modules/command.py)

        The `command' module takes the command name followed by a list of space-delimited arguments.
        The given command will be executed on all selected nodes. The command(s) will not be
        processed through the shell, so variables like `$HOSTNAME' and operations like `"*"', `"<"',
        `">"', `"|"', `";"' and `"&"' will not work. Use the [ansible.builtin.shell] module if you
        need these features. To create `command' tasks that are easier to read than the ones using
        space-delimited arguments, pass parameters using the `args' task keyword
        <https://docs.ansible.com/ansible/latest/reference_appendices/playbooks_keywords.html#task>
        or use `cmd' parameter. Either a free form command or `cmd' parameter is required, see the
        examples. For Windows targets, use the [ansible.windows.win_command] module instead.

ADDED IN: historical
......
```



##### debug – Print statements during execution

https://docs.ansible.com/ansible/2.9/modules/debug_module.html#debug-module

- This module prints statements during execution and can be useful for debugging variables or expressions without necessarily halting the playbook.
- Useful for debugging together with the `when:` directive.
- This module is also supported for Windows targets.

`playbook3.yaml`

```
- name: Hello World
  hosts: localhost

  tasks:
    - name: Hello World
      debug:
        msg: "Hello World from debug-module"
```

```
$ ansible-playbook playbook3.yaml
[WARNING]: No inventory was parsed, only implicit localhost is available
[WARNING]: provided hosts list is empty, only localhost is available. Note that the implicit localhost does not match 'all'

PLAY [Hello World] *************************************************************

TASK [Gathering Facts] ********************************************************************************
ok: [localhost]

TASK [Hello World] ********************************************************************************
ok: [localhost] => {
    "msg": "Hello World from debug-module"
}

PLAY RECAP ********************************************************************************
localhost                  : ok=2    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
```



---



##### Using Variables

https://docs.ansible.com/ansible/latest/playbook_guide/playbooks_variables.html

`playbook4.yaml`

```
- name: Hello World
  hosts: localhost

  vars:
    greetings: "Hello from vars"

  tasks:
    - name: Hello World
      debug:
        msg: "{{ greetings }}"
```

```
$ ansible-playbook playbook4.yaml
```



##### Defining variables in included files and roles

https://docs.ansible.com/ansible/latest/playbook_guide/playbooks_variables.html#defining-variables-in-included-files-and-roles

```
mkdir ~/ansible/playbook/vars
touch ~/ansible/playbook/vars/greeting_vars.yaml
```

`vars/greeting_vars.yaml`

```
---
ext_greetings: "Hello from ext vars"
```

`playbook5.yaml`

```
- name: Hello World
  hosts: localhost

  vars:
    greetings: "Hello from vars"

  vars_files:
    - "vars/greeting_vars.yaml"

  tasks:
    - name: Hello World
      debug:
        msg: "{{ ext_greetings }}"
```



---



##### Loops

```
touch ~/ansible/playbook/vars/esxi_hosts.yaml
```

`vars/esxi_hosts.yaml`

```
---
esxi_hosts:
  - "10.1.1.101"
  - "10.1.1.102"
  - "10.1.1.103"
```

`playbook6.yaml`

```
- name: Hello World
  hosts: localhost
  gather_facts: no

  vars:
    test:
    - test1
    - test2
    - test3

  vars_files:
    - "vars/esxi_hosts.yaml"

  tasks:
    - name: Loop demo
      debug:
        msg:
        - "{{ test }}"
        - "{{ esxi_hosts }}"
```

##### Iterating over a simple list

You can define the list in a variables file, or in the `vars` section of your play, then refer to the name of the list in the task.

```
loop: "{{ somelist }}"
```

`playbook7.yaml`

```
- name: Hello World
  hosts: localhost
  gather_facts: no

  vars:
    test:
    - test1
    - test2
    - test3

  vars_files:
    - "vars/esxi_hosts.yaml"

  tasks:
    - name: Loop demo
      debug:
        msg: "{{ item }}"
      # with_items: "{{ test }}" # Old syntax
      loop: "{{ test }}"
```

###### Registering variables with a loop

You can register the output of a loop as a variable.

When you use `register` with a loop, the data structure placed in the variable will contain a `results` attribute that is a list of all responses from the module. This differs from the data structure returned when using `register` without a loop.

`playbook8.yaml`

```
- name: Hello World
  hosts: localhost
  gather_facts: no

  vars:
    test:
    - test1
    - test2
    - test3

  vars_files:
    - "vars/esxi_hosts.yaml"

  tasks:
    - name: Loop demo
      debug:
        msg: "{{ item }}"
      loop: "{{ test }}"
      register:	test_result

    - name: Registering var with a loop demo
      debug:
        msg: "{{ item }}"
      loop: "{{ test_result.results }}"
```

