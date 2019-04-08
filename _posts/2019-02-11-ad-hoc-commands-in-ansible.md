---
  layout: post
  title: Ad-hoc Commands in Ansible
  subtitle:
  tags:
  categories:
  author: Eshan Shafeeq
  header_img:
---

## Overview
* What is ad-hoc commands
* Use cases for ad-hoc commands
* ad-hoc vs playbook
* Ansible command syntax
* Common modules

## What is an ad-hoc Ansible Command
* You can either run ansible either ad-hoc or as a playbook, both have the same
  capabilities.
* Ad-hoc commands are basically one-liners. You can run a single command or an
  ansible module with some parameters against a target host or a group of hosts in an inventory
  file. You will get feedback on how that command or module ran.
* Where as a playbook would be a series of tasks or modules with paramters get
  run against a group of hosts.

## Use cases for Ad-hoc
* Operational Commands
	* Checking log contents - *quick production checks*
	* Daemon control - *stop or start a particular daemon across few 100s
	  of servers*
	* Process management - *Kill a process within all the servers*
* Informational Commands
	* Check installed software on a particular host / group of hosts.
	* Check system properties.
	* Gathering system performance information (CPU, diskspace, memory
	  use).
* Research
	* Work with unfamiliar modules on test systems.
	* Practice for playbook engineering.

## AD-HOC VS Playbook


| Ad-hoc mode                                                                                 | Playbook mode                                                 |
| -----------------                                                                           | -------------------------                                     |
| command : ansible                                                                           | command: ansible-playbook                                     |
| effective for one-time commands, operational activities, information gathering and research | effective for deployments, routine tasks, system deployments. |
| similar to a single bash command                                                            | similar to a bash script                                      |

## Ansible Command Syntax

Example : installing apache


```shell
# httpd-servers group
# inventory file ansible/inv.ini
# -b is become to root user by default
# -m is the module, in this case yum module,
# -a is used to pass argument to the module,
# name is the name of the package,
# state is the version in this case latest.
# -f is the flag to fork, default is 5 (run the command against 5 servers at a
# time) here we are setting it to 100

ansible httpd-servers -i ansible/inv.ini -b -m yum -a "name-httpd state-latest"
-f 100
```

## Common Module

| Ping                                | Setup                  | Yum                     |
| ----------------------------------- | ---------------------- | ----------------------- |
| Validate server is up and reachable | Gather ansible facts   | Use yum package manager |
| No required parameters              | No required parameters | Name and State          |


| Service                             | User                    | Copy                    |
| ----------------------------------- | ----------------------  | ----------------------- |
| Control Daemons                     | Manipulate System users | Copy files              |
| name and state                      | name                    | src and dest            |


| File                                | Git                             |
| ----------------------------------- | ------------------------------- |
| work with files                     | Interact with git repositories  |
| path                                | repo and dest                   |


## Sample ad-hoc commands

```shell
# create a user supervisor all the systems in your hosts file
ansible all -m user -a "name=supervisor group=supervisor generate_ssh_key=yes"

# copy a file over to all the hosts
ansible all -m copy -a "src=/home/me/file dest=/home/supervisor/file"

# start auditd service
ansible all -m systemd -a "name=auditd state=started enabled=yes"
```
