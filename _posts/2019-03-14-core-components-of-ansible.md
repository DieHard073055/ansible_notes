---
layout: post
title: core components of ansible
subtitle: what you need to know before going out with ansible
gh-repo: DieHard073055/ansible
gh-badge: [star, fork, follow]
tags: [ansible]
comments: true
---
### Inventories
#### Static Inventory
Inventory refers to a file / ( an executable which returns JSON data ) that ansible uses to identify its target hosts. You would normally group the hosts into groups based off of how you would like to run ansible commands on them. Same host can be in multiple groups.

{% raw %}
By default ansible uses the `/etc/ansible/host` as the default inventory. If you wish to use a different inventory file you can specify the inventory file with the `-i` flag
```bash
ansible webservers -i inv/targets.ini -m yum -a "name=mariadb-server state=latest"
ansible-play -i inv/prod-targets.ini -m service -a "name=rabbitmq-server state=restarted"
```
example inventory file
```ini
[webservers]
web1.ansible.com
web2.ansible.com

[databaseservers]
db1.ansible.com
db2.ansible.com
```

If you have your hosts named with consecutive numbers. You could do something as follows.

```ini
[webservers-asia]
asia-web[1:20].ansible.com

[webservers-europe]
eruo-web[1:67].ansible.com
```
You also have the option to add varibles associated with the hosts in the inventory file.
```ini
[reverse-tunnel]
localhost ansible_host=localhost ansible_ssh_pass=EUF&n39dija ansible_port=43349
```
Ansible will use those variables during ssh. In order to see the rest of the variables available take a look at the [ansible user guide](https://docs.ansible.com/ansible/latest/user_guide/intro_inventory.html#list-of-behavioral-inventory-parameters).

#### Dynamic Inventory
These [days](days) we are normally using a cloud server to host our application. Ansible is run against a set of hosts which are spin up by our cloud service providers. Its sometimes convenient to hit an API and get our hosts. This is where dynamic inventories come into play. You can have an executable, any kind of executable, as long as it has the permissions to execute, and returns JSON. You can specify that executable as your inventory file with the `-i` flag, and it will work.

These executables *must* accept two flags which are required by ansible. `--list` and `--host` flags. The `--list` flag is used to list all the hosts, groups and child groups. `--host <hostname>` is passed to get any associated variables for the provided `<hostname>`.
```bash
$ ./inventory.py --list
{
    "webservers": {
        "hosts": ["web1.ansible.com", "web2.ansible.com"],
        "vars": {
            "http_port": 8080
        },
        "children": ["nodeservers"]
    },
    "nodeservers": {
        "hosts": ["web3.ansible.com","web4.ansible.com"],
        "vars": {
            "http_port": 5000
        },
        "children":[]
    }

}
$ ./inventory.py --host web1.ansible.com
{
    "http_port":8080
}
```
> an example expected behaviour from the script.

### Modules
This is what makes ansible. Modules provide way for ansible to interact with the services running on the targets. Modules are the code which performs the required action based on the plays you have defined in your playbooks.

#### Some of the common modules
- Ping Module
  - Allows you to ping the target host. To know if they are reachable.
  - `ansible all -m ping` to check if all the hosts in your inventory file are reachable.
- Setup Module
  - Allow you to gather all the facts about a specific target host.
  - `ansible web1.ansible.com -m setup -a "filter=ansible_enp2s0"` this will give you the information about the enp2s0 interface.
- Copy Module
  - Allows you to copy files from ansible-control node to ansible-target node, or within the ansible target node.
- Yum Module
  - Allows you to install packages in CentOS or RedHat environments.
- Shell Module
  - Allows you to run shell commands in a specific shell environment.
- Command Module
  - Allow you to run commands in general. Will not be able to configure the shell environment.
- Service Module
  - Allows you to start and stop systemd services.
  - `ansible web1.ansible.com -m service -a "name=nginx state=restarted"` will restart nginx on web1.ansible.com.
- Debug Module
  - Allows you to print variables and other messages on while executing ansible playbook tasks.

### Variables
Variables in ansible can be used to alter the behaviour of the execution. And most used to dynamically create configuration which cater the needs of the target host.
In order to use variables, ansible only accepts valid varible names. They have to start with letters, can contain numbers and `_`. Thats about it. You cannot have `-` or `.` or spaces in between words. And the variable cannot be a number either.
Your variables can be dictionaries that map to key values.
```yaml
webserver:
  username: web_admin
  password: Urf*@2187_Redds!
```

You can reference those values with the dot notation or bracket notation. Dot notation will not work if it collides with python public attributes or dunder methods and attributes. Its best practise to use bracket notation.
```
tasks:
- name: Show the username and password
  debug:
    msg: "{{webserver['username']}}:{{webserver['password']}}"
```
The double curly braces are jinja2 syntax. Ansible supports jinja2 syntax for both processing templates and strings.
You can use variables in conjunction with `when` keyword.
```yaml
tasks:
- name: Install apache for CentOS and RedHat
  yum:
    name: httpd
    state: latest
  when: (ansible_facts['distribution'] == "CentOS") or (ansible_facts['distribution'] == "RedHat")
  become: yes
```
Using `block` in conjunction with the `when` keyword can allow you to run a block of tasks after checking a condition as demonstrated above. And if the tasks failed it can run another set of tasks so that our playbooks can be more fault tolerant.
```yaml
tasks:
  block:
   - name: running a task which is definetly going to fail.
     command: /bin/false
   - name: Never executed
     debug:
       msg: ":("
  when: ansible_facts['distribution'] == "RedHat"
  rescue:
    - name: this will be called because the block failed to run
      debug:
        msg: ":)"
  always:
    - name: this task will be called always no matter what happens
      debug:
        msg: ":D"
```
Also you can use variables in your jinja2 templates.

### Facts
When you run ansible against a target system, ansible will collect information about that system and store them as facts. This is accessible via `ansible_facts` globally. Using the setup module you can gather facts from any host. There may be ocassions where you want to disable gathering facts.
```yaml
- hosts: databases
  gather_facts: no
```
Doing so will speed up the playbook execution process.
You can also have custom facts gathered on localhost.
##### /etc/ansible/facts.d/mongodb.fact
```ini
[general]
version: 3.4.3
size: 400000
```
If you have the file above in the specified location you should be able to read the values
```yaml
- hosts: local
  tasks:
  - name: Mongo version
    debug:
      msg: "{{ansible_local['mongodb']['version']}}"
```
> All facts will be accessible lower cased when using `ansible_local['lowercase_value']`.

You can also use the global variable `ansible_version` to define specific behaviour for ansible depending on the version you are running.

In order to get information about another host, Ansible would have had to gather facts on that target either on a previous task or play.
```
- hosts: db5.ansible.com
  tasks:
    - name: gather facts from db5
      setup:

- hosts: localhost
  tasks:
  - name: show the os family of db5
    debug:
      msg: "{{ hostvars['db5.ansible.com']['ansible_facts']['os_family'] }}"
```
It is also possible to do fact caching in between ansible playbook executions. If you are dealing with very large groups of servers. It will take alot of time to gather facts from all those instances. You can do this by installing redis and configuring ansible.cfg to use redis as cache.

### Plays & Playbooks
Ansible plays and playbooks are what makes Ansible. It is a thorough definition of how a system should be configured. If you organize your playbooks well, this can be used as the documentation of your infrastructure. At the same time use it to configure your infrastructure.
Ansible playbooks are YAML based configuration files. It normally consists of lists of tasks which needs to be run against a target host / host group.
#### update_package_cache.yml
```yaml
- hosts: us-east-webservers
  tasks:
  - name: update the package cache for all the web servers in US east region
    yum:
      update_cache: yes
    become: yes

```
Above is a simple playbook with just one play, and one task. however you can have multiple plays
#### install_httpd_and_postgres.yml
```yaml
# First play
- hosts: all-webservers
  # disable fact gathering, since we dont need it
  gather_facts: no
  # the user to use, when logging into the target host
  remote_user: web_admin
  # custom variables
  vars:
    http_port: 8080
  # task list to run for this play
  tasks:
  - name: Install the latest version of httpd
    yum:
      name: httpd
      state: latest
    become: yes
    handlers:
      - name: restart httpd
# Second Play
- hosts: all-databaseservers
  gather_facts: no
  remote_user: db_admin
  vars:
    db_connection_port: 5432
    db_root_user: pg_admin
  tasks:
  - name: Install the latest version of postgres in all the database servers
    yum:
      name: postgresql
      state: latest
    become: yes
    handlers:
      - name: restart postgres
```
By specifying `hosts` you are targeting the tasks below this to be run on that host. You can also include some custom variables under `vars`. If you dont need facts about the target hosts, you can disable facts gathering.
The goal when writing playbooks is to make them resuable. When you do attempt this, you will endup with a very large playbook. The next step would be to break this down into multiple files to make it manageable. This can be done using the `include_tasks` and `import_tasks`. You can also import playbooks using `import_playbook`.
```yaml
- hosts: webserver
  task:
    - name: Update package cache
      yum:
        update_cache: yes
- name: Import db playbook
  import_playbook: db_playbook.yml
```
Or you can just import tasks using `import`.
```yaml
- hosts: webservers
  gather_facts: yes
  tasks:
    - import_tasks: webserver_redhat.yml
      when: ansible_facts['distribution'] == "RedHat"
    - import_tasks: webserver_debian.yml
      when: ansible_facts['distribution'] == "Debian"
```
Include tasks can be used to achieve something similar.
```yaml
- hosts: IoT_edge-nodes
  gather_facts: yes
  vars:
    remote_user: admin
  tasks:
    - name: load the playbook depending on the device mac address
      include_tasks: "device_{{hostvars[inventory_host]['ansible_eth0']['ipv4']['macaddress']}}.yml"
```
There is a difference between import tasks and include tasks. `import_tasks` will be preprocessed at the time the playbooks are parsed. Normally use this when you know the task to import the task before hand. Like the example above. You would use `include_tasks` when you will know the name of task at runtime. The task file is parsed at runtime as well.

### Configuration Files
Ansible uses a configuration file to set settings. And these settings can be adjusted however you would like. Normally the configuration file is at `/etc/ansible/ansible.cfg`. Ansible looks for the configuration file in the following order:
- `ANSIBLE_CONFIG` environment variable.
- `ansible.cfg` in the current directory
- `~/.ansible.cfg` in the home directory
- `/etc/ansible/ansible.cfg` the default config which is added during ansible install.

You can use the options of providing ansible config to override depending on your need. The flexibility does come with security risks. If you leave it in a directory where it is wriable.
{% endraw %}
