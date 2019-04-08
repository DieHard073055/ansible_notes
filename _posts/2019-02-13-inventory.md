---
  layout: post
  title: Inventory
  subtitle:
  tags: [  ]
  categories:
  author: Eshan Shafeeq
  header_img:
---

# Inventory Essentails and Inventory Variables

### Overview
* Use both static and dynamic inventories to define groups of hosts:
    * What is the inventory
    * File formats
    * Static vs Dynamic
    * Variables and inventories
* Utlize an existing dynamic inventory script
    * On dynamic inventories
    * Some popular options

### What is an inventory
* An inventory is a list of hosts that ansible manages.
* Inventory location maybe specified as follows.
    * **Default :** /etc/ansible/hosts.
    * **Specified by cli :** ansible -i `<filename>`.
    * Can be set in ansible config.
* Inventory file can contain hosts, patterns, groups and variables.
* You may specify the inventory as a directory containing series of inventory files (both static and dynamic)
* The inventory maybe specified in YAML or INI format
* Can be static or dynamic

### File Format

#### YAML

```yaml
---
all:
    hosts:
        mail.example.com
            ansible_port: 5556
            ansible_host: 192.168.1.24
    childeren:
        webservers:
            hosts:
                httpd1.example.com
                httpd2.example.com
        vars:
            http_port:8000
        labservers:
            hosts:
                lab[01:99].example.com
...
```

#### INI

```ini
mail.example.com ansible_port=5556 ansible_host=192.168.1.24

[webservers]
httpd1.example.com
httpd2.example.com

[webserver:vars]
http_port=8000

[labservers]
lab[01:99].example.com
```

### Static vs Dynamic inventory

| Static | Dynamic |
|--------|---------|
| INI or YAML format | Executables ( Bash scripts / python scripts / etc ) |
| Maintained by hand | Script returns JSON containing the inventory information |
| Easy to manage for static configuration | Good for cloud resources which will be subject to sudden change |


### Variables and Inventories
* Ansible recommends that variables not be defined in inventory files.
* Should be stored in YAML files located relative to inventory files.
* File structure should be as follows:
	* inventory/
		* group_vars/
			* `<group name>`
		* host_vars/
			* `<host name>`
* Files named by host or group may end in yml or yaml

```ini
# inventory.ini

mail.example.com

[httpd]
httpd1.example.com
httpd2.example.com
```

> Dynamic inventories use an executable which can be a python script or a bash
script, As long as its an executable, we can pass it to ansible with the -i
flag.
