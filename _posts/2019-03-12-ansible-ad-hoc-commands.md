---
layout: post
title: ansible ad hoc commands
subtitle: example ansible adhoc commands
gh-repo: DieHard073055/ansible
gh-badge: [star, fork, follow]
tags: [ansible]
comments: true
---

### Running a ad-hoc command
```bash
ansible webservers -m ping
```
The example above will ping all the web servers in your `/etc/ansible/inventory`.

```bash
ansible webservers -m user -a "name=admin group=admin generate_ssh_key=yes ssh_key_bits=4096" --become
```
creates a user admin on all the web servers and generates an ssh key for the user.
And the command will be run as super user `--become`

```bash
ansible webservers -m file -a "path=/home/user4/script.sh mode=0555" --become --become-user user4 --ask-become-pass
```
This will create an executable empty file in `/home/user4/script.sh`. But it will do so as user4.
This will also ask the user4 password to authenticate the request.

```bash
ansible webserver -B 3600 -P 0 -a "/home/user4/script.sh --running-long-operation"
```
This will run the script `/home/user4/script.sh` in all the webservers and put the task in the background.
The task will timeout out in 3600s. And since we set polling to 0, there will be no polling.
If you want to see the update get the job_id which was returned from running the command, and use the
async_status module.

```bash
ansible web1 -m async_status -a "jid=[job id from execution]"
```
Running that command will give the status of the job running on web1 instance.

```bash
ansible all -m setup
```
Running the command above will return all the facts about all the hosts. You can use the filter argument
to filter for specific results.

```bash
ansible localhost -m setup -a "filter=ansible_*_mb"
```
This will return all the facts about the memory from localhost.


### Notification

{: .box-note}
**Note:** This is a notification box.

### Warning

{: .box-warning}
**Warning:** This is a warning box.

### Error

{: .box-error}
**Error:** This is an error box.
