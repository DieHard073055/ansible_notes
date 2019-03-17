---
layout: post
title: configure ansible control node
subtitle: Each post also has a subtitle
gh-repo: DieHard073055/ansible
gh-badge: [star, fork, follow]
tags: [ansible]
comments: true
---

### Installing the required packages
To get started with ansible, we need python and python-pip installed. Most linux distros comes with python and pip pre-installed. In case you want to install it grab it from [python.org](https://www.python.org/downloads/). Also please make sure you have ssh installed on all the machines. Even the target machines. Ansible uses python and ssh in conjunction to configure hosts. So make sure, `openssh-server` is installed on all the hosts and the openssh service is running. `sudo systemctl start openssh; sudo systemctl enable openssh`.

#### To check if you have pip installed
```bash
$ pip --version
pip 19.0.3 from /usr/lib/python2.7/site-packages/pip (python 2.7)
```
Once you made sure you have pip installed. You can install ansible using `sudo pip install ansible`.
#### Check if you have ansible installed
```bash
ansible localhost -m ping

 [WARNING]: Unable to parse /etc/ansible/hosts as an inventory source

 [WARNING]: No inventory was parsed, only implicit localhost is available

 [WARNING]: provided hosts list is empty, only localhost is available. Note that the implicit localhost does not
match 'all'

localhost | SUCCESS => {
    "changed": false,
    "ping": "pong"
}
```
Ansible is installed. One more package left to install get started with configuring the targets. That is `sshpass`. This allows us to ssh in to other machines and provide password to `ssh` programatically. On RedHat you would install is like so `sudo yum install -y sshpass`.

Now we have installed all the required packages!

### Set /etc/hosts

{: .box-note}
*Note:* This is an optional step to setup your `/etc/hosts`. I did this so i can use the name `targetnode` in my inventory files.

My current setup of `/etc/hosts` looks like so:
#### /etc/hosts
```ini
172.31.11.84    targetnode
```
Make sure you can ping your target node. And it is accessible to you via the network. Also make sure you can ssh into your target node from your control node.
```bash
$ ping -c 1 targetnode
PING targetnode (172.31.11.84) 56(84) bytes of data.
64 bytes from targetnode (172.31.11.84): icmp_seq=1 ttl=64 time=0.761 ms

--- targetnode ping statistics ---
1 packets transmitted, 1 received, 0% packet loss, time 0ms
rtt min/avg/max/mdev = 0.761/0.761/0.761/0.000 ms
```

Now on to creating a static inventory file.
### Creating a static inventory file
#### ~/targets.ini
```ini
targetnode  ansible_user='cloud_user' ansible_ssh_pass='SAJU1405htrp' ansible_become_pass='SAJU1405htrp' ansible_ssh_extra_args='-o StrictHostKeyChecking=no'
```
Run ansible against this `targetnode`
```
$ ansible all -i target.ini -m ping
targetnode | SUCCESS => {
    "changed": false,
    "ping": "pong"
}
```

{: .box-note}
*Note:* Make sure you have generated a pair of ssh public/private key using `ssh-keygen`.

### Configure targets
Now we can use this inventory file to configure the target hosts. Create the user ansible on the target node. Add the control node's ssh public key into the `/home/ansible/.ssh/authorized_keys` for all the target nodes.

we will add the password we are creating for the ansible user in an encrypted file, using the `ansible-vault`.
```bash
ansible-vault create passwordfile
```
For me, this opens up an empty file on vim (after i entered the encryption password). And then i added the following yml syntax.
#### ~/passwordfile
```yaml
password: mysecretpassword
```
I saved the encryption password to another file called vault-pass.
I used the following playbook to configure the ansible target hosts.
#### ~/configure_targets.yml
{% raw %}
```yaml
---
- hosts: targets
  tasks:
    - name: include secret password file
      include_vars: passwordfile
    - name: create user ansible
      user:
        name: ansible
        password: "{{password}}"
      become: yes
    - name: add control node to authorized keys
      authorized_key:
        user: ansible
        state: present
        key: "{{ lookup('file', '/home/ansible/.ssh/id_rsa.pub') }}"
        manage_dir: no
      become: yes
```
{% endraw %}
Run the play book using `ansible-playbook --vault-password-file=vault-pass -i targets.ini configure_targets.yml. That should configure all your targets for you to ssh into them without the credentials.

Now we can disable login with passwords. So only users with their public key in the authorized keys list can login to the targets.
```yaml
---
- hosts: targets
  tasks:
    - name: disable password on ssh
      lineinfile:
        path: /etc/ssh/ssh_config
        regexp: "PasswordAuthentication .*$"
        line: "PasswordAuthentication no"
      become: yes
    - name: restart ssh-server
      service:
        name: sshd
        state: restarted
      become: yes

```
Now the ansible control node is setup.
