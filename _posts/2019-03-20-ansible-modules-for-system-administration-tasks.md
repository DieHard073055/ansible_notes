---
layout: post
title: System Administration Tasks
subtitle: when you ask ansible to do chores
gh-repo: DieHard073055/ansible
gh-badge: [star, fork, follow]
tags: [ansible]
comments: true
---

Here we will go through examples of doing tasks with ansible.

### Overview
* Software Packages and Repositories
* System Services
* Firewall Rules
* File Content
* External APIs
* Managing the Scheduling of Tasks
* Managing System Security
* Managing System Users and Groups

### Software Packages and Repositories

This play book will about updating the package manager for RedHat ( yum ). And then enabling epel packages for RedHat. Which is the opensource free community based project from fedora team. Afterwards we will install ufw which is an uncomplicated firewall for linux distributions.
{% raw %}

```yaml
---
- hosts: labserver1
  vars:
    release_version: 7
    base_arch: x86_64
    epel_repo_url: "https://dl.fedoraproject.org/pub/epel/epel-release-latest-{{ ansible_distribution_major_version }}.noarch.rpm"
    epel_repo_gpg_key_url: "/etc/pki/rpm-gpg/RPM-GPG-KEY-EPEL-{{ ansible_distribution_major_version }}"
    epel_repofile_path: "/etc/yum.repos.d/epel.repo"
  tasks:
    - name: update yum package manager
      yum:
        update_cache: yes

    - name: import role to enable epel
      import_role:
        name: geerlingguy.repo-epel

    - name: check for epel repo configuration
      stat: path={{ epel_repofile_path }}
      register: epel_repofile_result

    - name: install epel repo
      yum:
        name: "{{ epel_repo_url }}"
        state: present
      register: result
      until: 'result.rc == 0'
      retries: 5
      delay: 10
      when: not epel_repofile_result.stat.exists

    - name: import epel GPG key
      rpm_key:
        key: "{{ epel_repo_gpg_key_url }}"
        state: present
      when: not epel_repofile_result.stat.exists
      ignore_errors: "{{ ansible_check_mode }}"

    - name: make sure ufw is installed and upto date
      yum:
        name: ufw
        state: latest
        disablerepo: "epel_repo"
      become: yes
```
### System Services
Since we have the firewall installed. We must make sure we have the firewall service running, and the service starts up the first thing when we reboot
```yaml
---
- hosts: labserver1
  tasks:
    - name: make sure ufw firewall service is up
      service:
        name: ufw
        state: started
      become: yes

    - name: make sure ufw firewall will be started when rebooting
      service:
        name: ufw
        enabled: yes
      become: yes
```
### Firewall Rules
We have gotton the firewall installed. We have the firewall service up and running. In case of system restart the firewall will be up automatically as well. Now to configure the firewall according to our needs.
```yaml
---
- hosts: labserver1
  vars:
   open_ports:
     - ssh
     - 443
     - 80
  tasks:
   - name: enable the firewall rules and reload ufw
     ufw:
       rule: allow
       port: "{{item}}"
       state: reloaded
     with_items:
       - "{{open_ports}}"
     become: yes
```
### File Content
In case you ever needed to lookup a file, load it into a variable and then maybe wanted to add the contents to a file in the remote server. Here's how you'd do something of similar sort.
```yaml
---
- hosts: labserver1
  vars:
    - file_location: /home/eshan/dev/ansible/system_admin/file1
    - contents: "{{ lookup('file', file_location)}}"
    - destination: ~/readme.txt
  tasks:
    - name: write the content read from the control node on to the target host
      copy:
        content: "{{contents}}"
        dest: destination

```

### External APIs
Everybody loves tacos! Whos up for tacos?! yep.. scroll down right this way.
```yaml
---
- hosts: localhost
  vars:
    num_recipes: 4
    file_name: []
  tasks:
    - name: create a directory for the recipes
      tempfile:
        state: directory
        suffix: recipe
      register: recipe_dir

    - name: create a directory for the recipe backup
      tempfile:
        state: directory
        suffix: recipe
      register: recipe_backup_dir

    - name: get random recipes
      uri:
        url: http://taco-randomizer.herokuapp.com/random/
      register: random_taco
      with_sequence: count={{ num_recipes }}

    - name: set file names based off of the recipe
      set_fact:
          file_name: "{{ file_name + [(recipe_dir['path'] + '/' + (random_taco['results'][item|int]['json']['base_layer']['name'] + ' with ' + random_taco['results'][item|int]['json']['mixin']['name'] + '.recipe') | regex_replace('\\s', '-')) ]}}"
      with_sequence: start=0 end={{ ( random_taco['results'] | length ) - 1}}

    - name: show recipe file locations
      debug:
          msg: "{{file_name[item|int]}}"
      with_sequence: start=0 end={{ ( random_taco['results'] | length - 1 ) }}

    - name: write the contents of the recepies to their files
      copy:
        dest: "{{file_name[item|int]}}"
        content: "{{( random_taco['results'][item|int]['json']['base_layer']['recipe'] + '\n' + random_taco['results'][item|int]['json']['mixin']['recipe'])}}"
      with_sequence: start=0 end={{ ( random_taco['results'] | length - 1 ) }}

    - name: create a backup of the recipes we have collected
      archive:
        path: "{{ recipe_dir['path'] }}"
        dest: "{{ recipe_backup_dir['path'] }}/recipe_backup.tgz"
```
{% endraw %}

The playbook above demostrates how you would hit a public api. Load the values into a variable. And also iterate through a dictionary and do checks on those variables. This is the equivalent of a for loop in python, or java. And finishes it off by archiving the saved recipe contents into another location.
