---
  layout: post
  title: Ansible Templates
  subtitle:
  tags: [  ]
  categories:
  author: Eshan Shafeeq
  header_img:
---

### Overview
* Template Basics
* Template Module
* Template File Format

### Template Basics
* Templates give the ability to provide a skeletal file that can be dynamically
  completed using variables
* The most common template use case is configuration file management
* Templates are generally used by providing a template file on the ansible
  control node, and then using the template module within your playbook to
  deploy the file to a target server or group.
* Templates are processed using the jinja2 template language

### Template Module

```yaml
---
- hosts: webservers

  tasks:

  - name: ensure apache is at the latest version
    yum: name=httpd state=latest

  - name: write the apache config file
    template: src=/srv/httpd.j2 dest=/etc/httpd.conf
```

* The template module is used to deploy the template files.
* There are two required parameters:
    * src - the template to use ( on the ansible control host )
    * dest - where the resulting file should be located ( on the target host )
* A useful optional paramter is validate which requires a successful validation
  command to run against the result file prior to deployment.
* It is also possible to set basic file properties using the template module (
  owner, permissions )

### Template File Format

```jinja2
{% raw %}
# Apache HTTP Server -- main configuration
#
#  {{ ansible_managed }}

    ## General Configuration
    ServerRoot {{ http_ServerRoot }}
    Listen {{ http_Listen }}

    Include conf.modules.d/*.conf

    User apache
    Group apache

    ## 'Main' server configuration
    ServerAdmin {{ http_ServerAdmin }}
    {% if httpd_ServerName is defined %}
    ServerName {{ httpd_ServerName }}
    {% end_if %}

    DocumentRoot {{ httpd_DocumentRoot }}

{% endraw %}
 ```

 * Template files are essentially a little more than text files.
 * Template files are designated with a file extention of .j2
 * Template files have the access to the same variables that the play that
   calls them does.

