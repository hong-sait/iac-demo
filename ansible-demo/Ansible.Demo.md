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
mkdir -p ~/ansible-demo/inventory
cd ~/ansible-demo/inventory
touch ~/ansible-demo/inventory/inventory.ini
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
mkdir -p ~/ansible-demo/playbook && cd ~/ansible-demo/playbook
touch ~/ansible-demo/playbook/playbook1.yaml
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

Create `inventory.ini` under `~/ansible-demo/playbook/`:

```
[web_srv]
ansible-node1 ansible_connection=ssh
ansible-node2 ansible_connection=ssh
```

```
$ ls -l ~/ansible-demo/playbook/
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
- name: module-demo
  hosts: web_srv
  tasks:
  - name: query hostname and related settings
    ansible.builtin.command: hostnamectl
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
mkdir ~/ansible-demo/playbook/vars
touch ~/ansible-demo/playbook/vars/greeting_vars.yaml
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
touch ~/ansible-demo/playbook/vars/esxi_hosts.yaml
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

##### Registering variables with a loop

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

##### Organizing host and group variables

https://docs.ansible.com/ansible/latest/inventory_guide/intro_inventory.html#organizing-host-and-group-variables

Store variables in the main inventory file: `inventory9.ini`

```
[web_srv]
ansible-node1 ansible_connection=ssh port=443
ansible-node2 ansible_connection=ssh port=443
```

Extract common vars out:

```
[web_srv]
ansible-node1
ansible-node2

[web_srv:vars]
ansible_connection=ssh
port=443
```

`playbook9.yaml`

```
---

- name: Hello World
  hosts: web_srv
  gather_facts: no

  tasks:
    - name: test vars
      debug:
        msg: "ansible_user={{ ansible_connection }} port={{ port }}"
```

```
ansible-playbook -i inventory9.ini playbook9.yaml
```

`inventory9-2.ini`

```
[web_srv]
ansible-node1 port=80
ansible-node2

[web_srv:vars]
ansible_connection=ssh
port=443
```

```
ansible-playbook -i inventory9-2.ini playbook9.yaml
```

Although you can store variables in the main inventory file, storing separate host and group variables files may help you organize your variable values more easily. You can also use lists and hash data in host and group variables files, which you cannot do in your main inventory file.

```
mkdir -p ~/ansible-demo/playbook/inventory/group_vars
mkdir -p ~/ansible-demo/playbook/inventory/host_vars
touch ~/ansible-demo/playbook/inventory/host

├── inventory
│   ├── group_vars
│   │   └── web_srv.yaml
│   ├── host
│   └── host_vars
│       └── ansible-node1.yaml
......
├── playbook9.yaml
```

`~/ansible-demo/playbook/inventory/group_vars/web_srv.yaml`

```
ansible_connection: ssh
port: 443
```

`~/ansible-demo/playbook/inventory/host_vars/ansible-node1.yaml`

```
port: 80
```

`~/ansible-demo/playbook/inventory/host`

```
[web_srv]
ansible-node1
ansible-node2
```

```
ansible-playbook -i inventory/host playbook9.yaml
```



---

##### ansible.cfg

https://docs.ansible.com/ansible/latest/reference_appendices/config.html

Ansible supports several sources for configuring its behavior, including an ini file named `ansible.cfg`, environment variables, command-line options, playbook keywords, and variables. See [Controlling how Ansible behaves: precedence rules](https://docs.ansible.com/ansible/latest/reference_appendices/general_precedence.html#general-precedence-rules) for details on the relative precedence of each source.

The `ansible-config` utility allows users to see all the configuration settings available, their defaults, how to set them and where their current value comes from. See [ansible-config](https://docs.ansible.com/ansible/latest/cli/ansible-config.html#ansible-config) for more information.

##### The configuration file

Changes can be made and used in a configuration file which will be searched for in the following order:

> - `ANSIBLE_CONFIG` (environment variable if set)
> - `ansible.cfg` (in the current directory)
> - `~/.ansible.cfg` (in the home directory)
> - `/etc/ansible/ansible.cfg`

Ansible will process the above list and use the first file found, all others are ignored.

```
touch ~/ansible-demo/playbook/ansible.cfg
```

`~/ansible-demo/playbook/ansible.cfg`

```
[defaults]
inventory = inventory/host
host_key_checking = False
```

Now the command line arg `-i inventory/host` can be omitted:

```
ansible-playbook playbook9.yaml
```



---

##### ansible.builtin.copy module – Copy files to remote locations

https://docs.ansible.com/ansible/latest/collections/ansible/builtin/copy_module.html

The [ansible.builtin.copy](https://docs.ansible.com/ansible/latest/collections/ansible/builtin/copy_module.html#ansible-collections-ansible-builtin-copy-module) module copies a file or a directory structure from the local or remote machine to a location on the remote machine. File system meta-information (permissions, ownership, etc.) may be set, even when the file or directory already exists on the target system. Some meta-information may be copied on request.

```
echo "demo" > ~/ansible-demo/playbook/demo.conf
```

`playbook10.yaml`

```
---

- name: ansible.builtin.copy module
  hosts: web_srv
  gather_facts: no

  tasks:
    - name: copy file
      ansible.builtin.copy:
        src: demo.conf
        dest: /home/ubuntu/demo.conf
        owner: ubuntu
        group: ubuntu
        mode: '0644'
```

```
ansible-playbook playbook10.yaml
ssh ansible-node1 cat /home/ubuntu/demo.conf
ssh ansible-node2 cat /home/ubuntu/demo.conf
```



---

##### Understanding privilege escalation: become

https://docs.ansible.com/ansible/latest/playbook_guide/playbooks_privilege_escalation.html

Ansible uses existing privilege escalation systems to execute tasks with root privileges or with another user’s permissions. Because this feature allows you to ‘become’ another user, different from the user that logged into the machine (remote user), we call it `become`. The `become` keyword uses existing privilege escalation tools like sudo, su, pfexec, doas, pbrun, dzdo, ksu, runas, machinectl and others.

##### ansible.builtin.apt module

https://docs.ansible.com/ansible/latest/collections/ansible/builtin/apt_module.html

Manages apt packages (such as for Debian/Ubuntu).

lighttpd is not installed on both servers under web_srv group:

```
ssh ansible-node1 systemctl status lighttpd
Unit lighttpd.service could not be found.
ssh ansible-node2 systemctl status lighttpd
Unit lighttpd.service could not be found.
```

`playbook11.yaml`

```
---

- name: Install lighttpd package on Ubuntu Linux servers
  hosts: web_srv
  become: true
  become_method: sudo
  gather_facts: no

  tasks:
    - name: Update repositories and install lighttpd package
      ansible.builtin.apt:
        name: lighttpd
        update_cache: true
        state: present
```

```
ansible-playbook playbook11.yaml
```

```
ssh ansible-node1 systemctl status lighttpd
● lighttpd.service - Lighttpd Daemon
     Loaded: loaded (/lib/systemd/system/lighttpd.service; enabled; vendor preset: enabled)
     Active: active (running) since Tue 2024-02-06 22:24:19 MST; 2min 18s ago
    Process: 4458 ExecStartPre=/usr/sbin/lighttpd -tt -f /etc/lighttpd/lighttpd.conf (code=exited, status=0/SUCCESS)
   Main PID: 4467 (lighttpd)
      Tasks: 1 (limit: 4535)
     Memory: 936.0K
        CPU: 271ms
     CGroup: /system.slice/lighttpd.service
             └─4467 /usr/sbin/lighttpd -D -f /etc/lighttpd/lighttpd.conf

Feb 06 22:24:19 ubuntu systemd[1]: Starting Lighttpd Daemon...
Feb 06 22:24:19 ubuntu systemd[1]: Started Lighttpd Daemon.
```

Visit the web servers to see the placeholder page. Create a local `index.html`:

```
echo "Peter Parker was here" > index.html
```

`playbook12.yaml`

```
---

- name: Install lighttpd package on Ubuntu Linux servers
  hosts: web_srv
  become: true
  become_method: sudo
  gather_facts: no

  tasks:
    - name: Update repositories and install lighttpd package
      ansible.builtin.apt:
        name: lighttpd
        update_cache: true
        state: present

    - name: Copy local index.html file to web servers
      ansible.builtin.copy:
        src: index.html
        dest: /var/www/html/index.html
        owner: ubuntu
        group: ubuntu
        mode: '0644'
```







