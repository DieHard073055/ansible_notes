---
  layout: post
  title: Ansible Variables & Facts
  subtitle:
  tags: [  ]
  categories:
  author: Eshan Shafeeq
  header_img:
---

### Overview
* Ansible Variables
* Dictionary Variables
* Magic Variables and filters
* What are facts?
* How to use facts
* Facts.d - custom facts

### Ansible Variables
* Naming ansible variables:
    * ansible variables can contain letters numbers and underscores.
* Some more places to define variables:
    * vars, vars_files, vars_prompt
    * Commandline: `ansible-playbook play.yml -e '{"date":"2019-08-04",
      "time":"08:26"}'`
    * Roles, blocks and inventories
* Essential Variable use:
    * - debug: msg="{% raw %} Look! I'm using my variable {{ myVar }}!{% endraw
      %}"
* A node on quotes:
    * Ansible may get confused about variables and module attributes if
      variables are not expressed with in a quote.
    * {% raw %} "{{ pakage }}" {% endraw %}

### Dictionary Variables
* Yaml formatting allows for python style dictonaries to be used as variables
* There are two formats to access dictionary values:
    * employee['name']
    * employee.name
* The bracket syntax is safer as the dot notation can have collisions with
  python in certain circumstances.

### Magic Variables and Filters
* Ansible defines several special variables known as magic variables.
* You can use the variable hostvars to look at facts about other hosts in the
  inventory.
  `{% raw %} {{hostvars['node']['ansible_distribution']}} {% endraw %}`
* There is also a groups variable that provides inventory information. `{% raw
  %}{{ groups['webservers'] }}{% endraw %}`
* Jinja2 filters can be useful in manipulating text format. `{% raw
  %}{{ groups['webservers'].join(' ') }}{% endraw %}`

Example of magic variable
```yaml
{% raw %}
---
- hosts: local
  vars:
    inv_file: /home/user/vars/inv.txt
  tasks:
    - name: create file
      file:
        path: "{{inv_file}}"
        state: touch
    - name:  generate inventory
      lineinfile:
        path: "{{inv_file}}"
        line: "{{ groups['labservers'].join(' ') }}"
{% endraw %}
```

The above example will generate a line of all the labservers defined in the
inventory file.

Another example
```yaml
# users.yaml
staff:
  - joe
  - john
  - bob
  - sam
  - mark
faculty:
  - matt
  - alex
  - frank
other:
  - will
  - jack
```


A playbook to interact with the text file above
```yaml
{% raw %}
---
- hosts: local
  vars:
    userFile: /home/user/vars/list
  tasks:
    - name: create file
      file:
        path: "{{ userFile }}"
        state: touch
    - name: list users
      lineinfile:
        path: "{{ userFile }}"
        line: "{{ item }}"
      with_items:
        - "{{ staff }}"
        - "{{ faculty }}"
        - "{{ other }}"
{% endraw %}
```

run the following command to see the playbook above in action
ansible-playbook userList.yml -e "@users.lst"

### What are facts?
* Facts are information discovered by ansible about a target system.
* There are two ways facts are collected:
    * Using the setup module with an ad-hoc command: ansible all -m setup
    * Facts are gathered by default when a playbook is executed
* Fact gathering in playbooks may be disabled using the gather_facts attribute.

### How to use facts?
* Any collected facts, they may be accessed through variables:
    * ansible_default_ipv4.address
* It is possible to use filters with regex, in adhoc mode, to match certain
  fact names.
* Facts may also be used with conditionals to have plays behave differently on
  hosts that meet certain criteria.

### Custom Facts [ facts.d ]
* It is possible to define custom facts on your servers using the facts.d
  directory.
* Create /etc/ansible/facts.d directory on the target system. All valid files
  within this directory ending in .fact are returned under ansible_local with
  facts
* Fact files may be INI, JSON, or an executable that returns JSON.

example custom fact file place it in the target system
```fact
# /etc/ansible/facts.d//prefs.fact
[location]
type=physical
datacenter=Alexandra
```

now on your control host run the ad-hoc command.
```ansible targetmachine -m setup -a "filter=ansible_local"```

