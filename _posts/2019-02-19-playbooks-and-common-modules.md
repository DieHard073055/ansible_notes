---
layout: post
title: Playbooks and Common Modules
---

### Overview
* Common ansible modules.
* Create playbooks to configure systems to a specified state.
* Use variables to retrieve the results of running commands.
* Use conditionals to control the play execution.
* Configure error handling.
* Selectively run specified tasks in playbooks using tags.

### Common Ansible Modules

* (Working with files) copy, archive, unarchive, get_url.
* user, group.
* ping.
* service.
* yum

#### Other useful modules
* lineinfile.
	* edit a specific line in a file..
	* if the line is present in the file it wont duplicate the line.
* htpasswd.
	* allows you to work with the htpasswd file.
* shell and command modules.
	* shell module helps you setup environment before running a command on
	  the machine.
	* command module will just execute the command without setting up the
	  environment.
* script module.
	* specifically designed for you to run scripts on the remote host.
	  takes a script which is to be run on the target host, sets up the
	  environment and runs the script.
* debug module.
	* it allows you to monitor the performance of a perticular module
	  youre running. You can print out the contents of variables to stdout

### Creating Playbooks to Configure Systems to a Specified State
```yaml
# demo.yaml
---
- hosts: labservers
    become: yes
    tasks:
        - name: install apache
          yum : name=httpd state=latest
        - name: start and enable httpd
          service: name=httpd state=started enabled=yes
        - name: create index.html
          file: path=/var/www/html/index.html state=touch
        - name: add a line to index.html
          lineinfile: path=/var/www/html/index.html line="<h1>Hello
          World!~</h1>"
...
```
> make sure you have your inventory file specified in /etc/ansible/hosts

```shell
# Run the play book
ansible-playbook demo.yml
```
now all your lab servers should have httpd installed, running and enabled.


### Using Variables to Retrieve the Results of a Running Command

* The register keyword
    * used to register variables within a play.

example playbook

```yaml
    tasks:
        - name: copy a file
          copy:
            src: testfile
            dest: /home/user/testregister
            mode: 400
          register: var
        - name: output debug info
          debug: msg="debug info is {{var}}"
```

another example playbook

```yaml
---
- hosts
  tasks:
    - name: create file
      file:
        path: /tmp/newfile
        state: touch
      register: output
    - debug:
      msg: "Register output is {% raw %}{{output}}{% endraw %}"
    - name: edit file
       lineinfile:
            path: /tmp/newfile
            line: {% raw %}"{{output.uid}}"{% endraw %}
...
```

### Use conditionals to control play execution

example of using handlers

```yaml
- name: template configuration file
  template:
    src: template.j2
    dest: /etc/foo.conf
  notify:
    - roll web
```

```yaml
handlers:
    - name: restart apache
      service:
        name: apache
        state: restarted
      listen: "roll web"
```


when ever a notify is invoked, the handler that is listening on that notify
topic will run.

#### other conditional keywords
* when
* with_items
* with_files

with_items example
```yaml
---
# this play book creates 3 users for us
- hosts: local
  become: yes
  tasks:
    - name: create user
      user:
        name: "{% raw %}{{item}}{% endraw %}"
      with_items:
        - sam
        - john
        - bob
...
```

when example
```yaml
---
- hosts: labservers
  become: yes
  tasks:
    - name: edit index
      lineinfile:
        path: /var/www/html/index.html
        line: "I'm Back!"
      when:
        - ansible_hostname == "me"
...
```

### Configure Error Handling
* Ignoring acceptable errors.
* Defining failure conditions.
* Defining "Changed"
* Blocks ( try/catch )

example of error handling
```yaml
---
# ignore errors
- hosts: local
  tasks:
    - name: get files
      get_url:
        url: "http://{% raw %}{{item}}{% endraw %}/index.html"
        dest: "/tmp/{% raw %}{{item}}{% endraw %}"
      ignore_errors: yes
      with_items:
        - server1
        - server2
        - server3
...
```

```yaml
---
# blocks to handle errors
- hosts: local
  tasks:
    - name: get file
      block:
          get_url:
            url: "http://server1/index.html"
            dest: "/tmp/index_file"
      rescue:
        - debug: msg="The file dosen't exist"
      always:
        - debug: msg"play is done"
...
```
in the above example, if the task fails it will run the rescue block and the
always block. Normally if it was successful it will run the task and the always
block.

### Selectively Run Specified Tasks in Playbook using Tags
* How ansible uses tags
    * allows you to run a set of tasks with a specific tag.

```yaml
# deploy.yaml
---
- host: web
  become: yes
  tasks:
    - name: deploy app binary
      copy:
        src: /home/user/apps/hello
        dest: /var/www/html/hello
      tags:
        - webdeploy
- host: db
  become: yes
  tasks:
    - name: deploy db script
      copy:
        src: /home/user/apps/scripts.sql
        dest: /opt/db/scripts/scripts.sql
      tags:
        - dbdeploy
...
```

if you want to only run the web deploy task

```
ansible-playbook deploy.yaml --tags webdeploy
```

if you want to only run the db deploy task

```
ansible-playbook deploy.yaml --tags dbdeploy
```

you can skip tags using the skip-tags switch

