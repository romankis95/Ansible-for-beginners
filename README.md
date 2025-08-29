# Ansible for beginners

My journey into learning ansible for automation. 

## Objectives

- Introdution to ansible
- Installation and configuration
- Ansibe Intenvory
- Ansible Playbooks and yaml
- Ansible Modules & plugins
- Ansible handlers
- Ansible Templates
- Resuing Ansible Content

---

### Introdution to ansible

Automating in IT requires coding skills, time and maintenance. Thats why ansible was made. It's simple, powerful and agentless.
It converts big scripts in few lines of an ansible playbook. 

Imagine you have a bunch of hosts that you want to restart in a particular order and time. Some of them are webservers, file servers, dbs etc.
In a real scenario you have to turn of the webservers, then the dbs and finally the file servers. 
The power on the file servers, dbs and finally the web servers. 

Another example:
Thanks to the built-in modules you can actually spawn multiple vms on private and public clouds. Manage the comunication between them, install application, creating users and managing firewall in no time. 

You can use existing services as DBS for inventories or even enabling tools such as ServiceNow to trigger ansible playbooks over actions. 

### Ansible configuration files

When you install ansible, it creates a defaulc configuration file at `/etc/ansible/ansible.cfg`. The ansible configuration file governs the default behaviour of ansible. 
An ansible configuration file is divided in several sections. For example the `[default]` section focus on there the inventory, the logs, library, roles etc are stored. 
For example if you run `cat ansible.cfg` you should get something like this:

```ini
[defaults]
inventory      = inventories/static/hosts.ini   
roles_path     = ./roles
stdout_callback = yaml

[inventory]
cache          = True                 
cache_plugin   = jsonfile
;cache_timeout  = 3600
....
```

In the `inventory` section you have the option to enable plugins and other settings that helps us to retrieve all the information we need. Btw we dont need right now to understand the options, we will cover them as soon as we start the relative topics. 
The values that we see in this configuration file will be considered by ansible if you run any playbook. But you can actually override them with dedicated options. For example, if i have different setups for my playbooks, such as dedicated for the DBs or Web servers or networking, I'll need a dedicated configuraiton file. Thats why you can just copy the default config file into separate folders and make any local adjustments you need. 

So when you will run the a playbook from these folders, ansible will pick the configuration file within them.

What if you wanted to store the configuration file in a commonn directory between playbooks? You can setup an enviriment variables such as `$ANSIBLE_CONFIG=/path/to/file.cfg`. What if we have all of them configured? In which order Ansible will consider the configuration files?
- Enviroment variable
- ansible.cfg in the current directory
- in user directory there can be an .ansible.cfg file
- default ansible.cfg file in `/etc/ansible`

These values dont have to have all the parameters, only the one you waht to override. Other values will be picked from the next file in the priority chain. 

If you wanted to override a single parameter, for instance `gathering   = implicit` you could define an env var only for that parameter. To do it you should first transform it to uppercase and append an "ANSIBLE_" to it. For example: `ANSIBLE_GATHERING=explicit`. It works for most of the configuration file paramether. And dont forget to use `export` to set the env var. 

If you run something like `$ ANSIBLE_GATHERING=explicit ansible-playbook playbook.yml` it will be used only for that single execution. But thats linux behaviour and not ansibles. 

You can run `$ ansible-config list` to view all the config files that are taken in accounts,  or run `$ ansible-config view` to see which config file is currently active. 
Using `ansible-config dump` you can see a comprehensive settings of current config and where ansible picks that from. 

### Ansible playbooks and yaml

All ansible files are written in yaml. Yaml separates key:value data. Remember you need to have a colon followed by a space between key and value, eg: `fruit: Apple`.
**Arrays** are rappresentented in the folloing way:

```yaml
Fruits:
- Orange
- Banana
- Apple

Vegetables:
- Carrot
- Tomato
```

How about a **dictionary**:

```yaml
Banana:
    Calories: 62
    Fat: 0.3
    Carbs: 16

Grape:
    Calories: 42
    Fat: 0.1
    Carbs: 11     
```

The **dictionary** has to have an equal namber of spaces at the beginning of the property. In this example it's like we had the following syntax: `Banana.Calories: 62` or `Banana.Fat: 0.3`. What if we mismatch some spaces?

```yaml
Banana:
    Calories: 62
     Fat: 0.3
     Carbs: 16
```

In this example our syntax is broken to this `Banana.Calories.Fat: 0.3` which is not the desired output. It also may trigger an syntax error because "Calories" has already a value assigned and it can't be a dictionary as well.

Please remember that the order of properties in a dictionary doesn't really matter as log as it's correctly formatted. It does matter in an array. 

### Laboratory 1
It's based on multiple choise questions and correcting some broken yaml. 

### Ansibe Intenvory

Ansible can work with or multiple systems in out infrastructure at the same time. In order to work with multiple servers, ansible needs to establish connectivity with those servers. This is done using ssh for linux or powershell remoting (winrm)for windows, these 2 methods are the reason why ansible is defined **agent-less**. 
Ansible needs some information about the hosts it has to connect to, thats the reason we have to create an inventory file.
By default ansible creats an inventory file located at `/etc/ansible/hosts` in **ini** format. 
For example:

```ini
[all]
server1.domain.lcl
server2.domain.lcl
server3.domain.lcl
server4.domain.lcl
server5.domain.lcl
server7.domain.lcl

[mail]
server3.domain.lcl
server4.domain.lcl

[db]
server5.domain.lcl
server7.domain.lcl
```
We can also extend this notation by assigning aliases to the servers, for example:

```ini
web ansible_host=server3.domain.lcl
db  ansible_host=server4.domain.lcl
```

We could extend further the inventory file by providing additional information:

```ini
web ansible_host=server3.domain.lcl ansible_connection=ssh ansible_port=22 ansible_user=srv_ansibile ansible_ssh_pass=dontdothat
db  ansible_host=server4.domain.lcl ansible_connection=winrm ansible_port=5985 ansible_user=srv_ansibile  ansible_password=dontdothat
```

We saw so far that the inventory can be written in `ini` format, but it can also be written in yaml as well. The yaml format as way better for larger scale projects. 

### Laboratory 2
Example of inventory created during the lab

```ini
# Web Servers
web1 ansible_host=server1.company.com ansible_connection=ssh ansible_user=root ansible_ssh_pass=Password123!
web2 ansible_host=server2.company.com ansible_connection=ssh ansible_user=root ansible_ssh_pass=Password123!
web3 ansible_host=server3.company.com ansible_connection=ssh ansible_user=root ansible_ssh_pass=Password123!

# Database Servers
db1 ansible_host=server4.company.com ansible_connection=winrm ansible_user=administrator ansible_password=Password123!

#web servers
[web_servers]
web1
web2
web3

[db_servers]
db1

[all_servers:children]
web_servers
db_servers
```

### Ansible variables

Variables same as in any other programming languages stores information. We already saw some variables before in the inventory files, e.g. `ansible_connection`. To use defined variables just enclose them in `{{variable_name}}`. You can have variables directly embedded inside a playbook, or inside inventory file or even better in a dedicated host file called hostname.yml in the same directory as the playbook. 

Variables can be **strings** , **numbers** , **booleans** , **lists** or **dictionaries**

```yml
username: "admin"
max_connections: 1000
delete: false
```

If you define variables at host level and a group level in inventory files, the ones at host level wins.

```ini
# Web Servers
web1 ansible_host=server1.company.com  
web2 ansible_host=server2.company.com ansible_connection=ssh ansible_user=root 
web3 ansible_host=server3.company.com 

[web_servers]
web1
web2
web3

[web_servers:vars]
ansible_connection=winrm
ansible_user=administrator
```
For the host `web2` the user will be root and the connection type ssh.

But, whathever i define at playbook level wins over host level variables. 
Once again if i run a playbook using cli and i pass the `extra-vars` argument. That variable wins over playbook level. 

```bash
$ ansible-playbook playbook.yml --extra-vars "ansible_user=pippo"
```
These ones are the most important. Please refer to the offical documentation for more information.

When creating a playbook with multiple tasks and onee task depeends on the output of the above one just use the argument `register: variable_name`. Now we can use variable_name in the tasks below. Please not that every register value depends on the module used. 

If we run a playbook that makes a cat of /etc/hosts file we need to parse the output first.

```yml
---
- name: get hosts
  hosts: all
  tasks:
  - shell: cat /etc/hosts
    register: result
  - debug: 
      var: result.stdout
```
What happens if a host wants to access variables defined for another host? Welp, it can't. But we can bypass this issue while writing a playbook using a magic variable, such as:

```yml
---
- name: get hosts
  hosts: all
  tasks:
  - debug: 
      msg: "{{hostvars['web2'].ansible_user}}"
```

### Ansible Facts

When we run a playbook and when ansible connects to a machine it first gathers some information regarding the host, such as architecture, network connectivity, interfaces, ips, mounts, volumes etc.

These information are known as `facts`.

These facts are collected using the `setup module`. This module is run automatically by ansible when you run a playbook even if not explicitly declared in the module. The following playbook runs 2 tasks and not one. The first one is silent and gather facts about the hosts. The second one execute the message. 

```yml
---
- name: Print hello world
  hosts: all
  tasks:
  - debug: 
      msg: Hello world
```

All facts gathered by ansible are stored in a variable called `ansible_facts`. If we wanted to, we could see the content of that variable with the following playbook.

```yml
---
- name: Print hello world
  hosts: all
  tasks:
  - debug: 
      var: ansible_facts
```

If we don't want to gather facts we need to declare it explicitly in the playbook with the property `gather_facts: no`.

```yml
---
- name: Print hello world
  hosts: all
  gather_facts: no
  tasks:
  - debug: 
      var: ansible_facts
```

You can globally define this property inside an ansible config file. 

### Laboratory 3

Variables from inventory file
```yml
---
- name: 'Add nameserver in resolv.conf file on localhost'
  hosts: localhost
  become: yes
  tasks:
    - name: 'Add nameserver in resolv.conf file'
      lineinfile:
        path: /tmp/resolv.conf
        line: 'nameserver {{  nameserver_ip  }}'
    - name: 'Disable SNMP Port'
      firewalld:
        port: '{{snmp_port}}'
        permanent: true
        state: disabled
```
Variables at playbook level
```yml
---
- hosts: localhost
  vars:
    car_model: 'BMW M3'
    country_name: USA
    title: 'Systems Engineer'
  tasks:
    - command: 'echo "My car is {{ car_model }}"'
    - command: 'echo "I live in the {{ country_name }}"'
    - command: 'echo "I work as a {{ title }}"'
```

Working with dictionaries

```yml
---
- hosts: all
  become: yes
  tasks:
    - name: Set up user
      user:
        name: "{{user_details.username}}"
        password: "{{user_details.password}}"
        comment: "{{user_details.email}}"
        state: present
```

### Ansible Playbooks

Plabooks are ansible's orchestartion language or better: playbooks do what we want ansible to do for us.
It can be simple as rebooting some vms or complex as deploying 50 vm in cloud, provision them, install apps, update them etc.

All playbooks are written in yaml language. A playbook is a set of tasks. Every task is an action that has to be performed on the host. Here is a simple playbook that we will analyze now:

```yml
---
- name: Play 1
  hosts: localhost
  tasks:
    - name: execute command date
      command: date
    - name: execute script on server
      script: script1.sh
    - name: install httpd service
      yum:
        name: httpd
        state: present
    - name: start webserver
      service:
        name: httpd
        state: started 
```

The goal of this playbook is to run is to run a set of activities, one after another on the localhost. Please note that the host that we want to run the playbook on is defined at playlevel. 
We can just edit `localhost` to anything present in the inventory file.

Fist we print the date, then we run a script on localhost, after that we install the httpd package and finally we start the httpd service and check if the current service status is equal to started. 

What happens if we split the tasks in 2 different plays in the same file?

```yml
---
- 
  name: Play 1
  hosts: localhost
  tasks:
    - name: execute command date
      command: date
    - name: execute script on server
      script: script1.sh

- 
  name: Play 2
  hosts: localhost
  tasks:
    - name: install httpd service
      yum:
        name: httpd
        state: present
    - name: start webserver
      service:
        name: httpd
        state: started 
```

The single `-` denotes that a playbook is a list of dictionaries. Each play is a dictionary. A task, on the contrary, is a list. You cant swap a task position with another task. Remember: a dictionary is an unordered collection, while a list is an ordered one. 
If you wanted to run the playbook on something different than localhost, you should first define that host in the inventory file, same goes for connections and other variables. If you play the playbook for `hosts: all` it will run on every host defined in the inventory file. 

The different actions done by the playbook are done through `modules`; in the previous playbook we used `command | script | yum | service` modules.

When you finally built your first ansible playbook, execute it using `$ ansible-playbook playbook.yml` command. 

### Verifying an Ansible playbook

Lets say that you need to verify that your playbook works accordingly before using it on production servers. Ansible provides several modes to verify playbooks: `check` mode and `diff` mode.

The check mode is a dry run; ansible runs the playbook without modifying anything on the machine. It allows the user to see the changes that will be made without applying them. To run the playbook in this mode you can append `--check` to the ansible playbook command. 
For example `$ ansible-playbook install_nginx.yml --check`.

Another mode is the diff mode.  It shows the before and after comparison of playbook changes. To run the playbook in this mode you can append `--diff` to the ansible playbook command. 
For example `$ ansible-playbook install_nginx.yml --diff`. You can also run diff and check together. 

We also have a syntax check mode. This mode checks if a playbook is syntax error free. To run the playbook in this mode you can append `--syntax-check` to the ansible playbook command. 
For example `$ ansible-playbook install_nginx.yml --syntax-check`. 

There is also `$ ansible-lint` that checks your playbooks for bugs, errors, suspicious constructions and helps you to maintain consistency quality beetween your plays. 

### Laboratory 4

How many playbooks are in the following yaml? How many tasks the Setup apache playbook has?

```yml
---
- name: Setup apache
  hosts: webserver
  tasks:
    - name: install httpd
      yum:
        name: httpd
        state: installed
    - name: Start service
      service:
        name: httpd
        state: started

- name: Setup tomcat
  hosts: appserver
  tasks:
    - name: install httpd
      yum:
        name: tomcat
        state: installed
    - name: Start service
      service:
        name: tomcat
        state: started
```

How to run this playbook in check mode?

```yml
- hosts: all
  tasks:
    - name: Install a new package
      apt:
        name: new_package
        state: present

    - name: Update the service
      service:
        name: my_service
        state: restarted

    - name: Check service status
      service:
        name: my_service
        state: started
```

How to run this playbook in diff mode? And how to run it in syntax-check mode?

```yml
- hosts: all
  tasks:
    - name: Set max connections
      lineinfile:
        path: /etc/postgresql/12/main/postgresql.conf
        line: 'max_connections = 500'

    - name: Set listen addresses
      lineinfile:
        path: /etc/postgresql/12/main/postgresql.conf
        line: 'listen_addresses = "*"'
```

How to run ansible lint on this playbook? What are the erros shows after running the linter?

```yml
- name: Database Setup Playbook
  hosts: db_servers
  tasks:
    - name: Ensure PostgreSQL is installed
      apt:
        name: postgresql
        state: latest
        update_cache: yes

    - name: Start PostgreSQL service
      service:
        name: postgresql
        state: started

    - copy:
        src: /path/to/pg_hba.conf
        dest: /etc/postgresql/12/main/pg_hba.conf
      notify:
        - Restart PostgreSQL
```
### Ansible Conditionals

In this chapter we will look into conditionals. Let's start with a simple example:
What if I had to install nginx on a debian machine and on a rhel one? As we know debian uses apt and rhel uses yum by default. 

This means that we have to write 2 playbooks, or write a single playbook with a condition on the task to run:

```yml
- name: Install NGINX
  hosts: all
  tasks:
    - name: Install nginx on debian
      apt:
        name: nginx
        state: present
      when: ansible_os_family == "Debian" and ansible_distibution_version == "16.04"
    - name: Install nginx on rhel
      yum:
        name: nginx
        state: present
      when: ansible_os_family == "RedHat" or ansible_os_family == "SUSE"
```

When a condition is true that task will run. Make sure to use `==` when checking for a condition to be equal. You can append more conditions using `and` or `or` operators. We can also do conditions in loops. 

```yml
- name: Install Software
  hosts: all
  vars:
    packages:
      - name: nginx
        required: True
      - name: mysql
        required: True
      - name: apache
        required: False
  tasks:
    - name: Install "{{item.name}}" on Debian
      apt:
        name: "{{item.name}}"
        state: present
      loop: "{{packages}}"
      when: item.required == True
```

Another way to use conditionals is based on previous tasks results. Let's create a playbook that checks a service status and if its down, sends an email.

```yml
- name: Monitor and alert
  hosts: node01
  tasks:
    - name: Check nginx
      command: service nginx status
      register: result
    - name: Send email
      mail:
        to: a@a.com
        subject: alert bla bla
        body: service nginx is down
        when: result.stdout.find('down') != -1
```

We can use ansible facts as variables in our conditional playbooks. For example if I needed to installa specific nginx package based on a distribution, I could gather facts and based on them write a condition like:

```yml
- name: Install nginx
  hosts: all
  tasks:
    - name: nginx=1.18.0
      state: present
      when: ansible.facts['os_family'] == 'Debian' and ansible_facts['distribution_major_version'] == '18'
```

### Lavoratory 5

```yml
---
- name: 'Add name server entry if not already entered'
  hosts: localhost
  become: yes
  tasks:
    - shell: 'cat /etc/resolv.conf'
      register: result
    - shell: 'echo "nameserver 10.0.250.10" >> /etc/resolv.conf'
      when: result.stdout.find('10.0.250.10') == -1
```

### Ansible Loops

Loops is a directive that executes the same task multiple times. Each time it runs it stores the value of each item in a loop variable.
Let's loook at the following playbook to understand it better:

```yml
---
- name: Create users
  host: all
  tasks:
    - user: name="{{ item  }}" state: present
      loop:
        - joe
        - nick
        - tean
        - mike
        - paul
```

Another way to create loops is using the `with` keyword: 

```yml
---
- name: Create users
  host: all
  tasks:
    - user: name="{{ item  }}" state: present
      with_items:
        - joe
        - nick
        - tean
        - mike
        - paul
```

It's recommended to use the lopp directive. With is an older directive that has a lot of variants.

### Laboratory 6

```yml
---
-  name: 'Print list of fruits'
   hosts: localhost
   vars:
     fruits:
       - Apple
       - Banana
       - Grapes
       - Orange
   tasks:
     - command: 'echo "{{ item }}"'
       with_items: '{{ fruits }}'
```

```yml
---
- name: 'Install required packages'
  hosts: localhost
  become: yes
  vars:
    packages:
      - httpd
      - make
      - vim
  tasks:
    - yum:
        name: '{{ item }}'
        state: present
      with_items: '{{ packages }}'
```

### Ansible Modules

Modules in ansible are categorized into various groups based on their functionality. For example:
- System
- Commands
- Files
- Database
- Cloud
- Windows

`System` modules are the one that can perform actions at system level, such as modifying the users, creating groups, editing ip tables, mouting volumes etc. The `commands` module is the one that allows us to run scripts or execute complex scripts on the hosts. `File` modules allows us to work with files, setting permisions such as ACL information. Archive files, replacing files etc.
`Database` modules help us working with databases such as mysql, mongodb, postgress.  `Cloud` modules help you to deploy resources to the cloud. The official ansible documentation provides useful information for each of the modules that we talked about and many more. 

Let's dig deeper with the `command` module: The command module is used to execute commands on the remote hosts. At the following url [Command Module](https://docs.ansible.com/ansible/latest/collections/ansible/builtin/command_module.html) we can find the official documentation for the command module. Let's write a simple playbook and start from there:


```yml
---
-  name: 'Execute commands on localhost'
   hosts: localhost
   tasks:
      - name: Execute command 1
        command: 'date'
      - name: Execute command 2
        command: 'cat /etc/resolv.conf'
      - name: Execute command 3
        command: 'cat resolv.conf chdir=/etc'
      - name: Execute command 4
        command: 'mkdir /test creats=/test'   
```

This playbook runs different commands on the localhost machine. 

The difference betweek the task 2 and 3 is that we used `chdir` parameter to be sure that andible changes the directory to `etc`. This is how parameters are passed to the official `command` module. The forth tasks ensures that a folder is not created if it's already present. 


Another module to look at is the `script` module. The script module allow us to run a script on the remote host after transferring it from the local ansible controller machine. 

A simple script module playbook look like this:

```yml
---
-  name: 'Play 1'
   hosts: node01
   tasks:
      - name: Run a script on the remote server
        script: /some/local/script.sh -arg1 -arg2 
```

Now let's take a look a the `service` module. The service module is used to maintain services in the desired state (started, active, stopped, restarting). The following ansible playbooks is used to start a variety of services:

```yml
---
-  name: 'Play 1'
   hosts: node01
   tasks:
      - name: Start the database service
        service: name=postresql state=started
      - name: Start the htppd service
        service: name=httpd state=started
      - name: Start the nginx service
        service: name=nginx state=started
```

As you can see using the `service` module we passed 2 parameters: name and the desired state of that module. 

We can also write the same playbook as follows:

```yml
---
-  name: 'Play 1'
   hosts: node01
   tasks:
      - name: Start the database service
        service:
          name: postresql
          state: started
          name: httpd
          state: started
          name: nginx
          state: started
```

Which to be honest i prefer. But why the action (state) is `started` and not `start`? 
If we were tasked to tell ansible to start a service named httpd we would say: `start the server httpd`. We are not instructing ansible to start a service, we are asking ansible to ensure that the httpd service is in the started state. 
If the service is already started, ansible should do nothing. This is colled `idempotency` and is is the core of all the ansible concept. As per the ansible documentation: **An operation is idempotent if the result of performing it once is exactly the same result as of performing it repetetly without any intervening actions**. Unfortunately not all the modules or commands are idempotent so be careful when reading the documentation regarding modules and actions. 

Finally lets look at the `lineinfile` module. This modules is used to look for a line in a file. Useful to check if `hosts` file is well populated etc.

```yaml
---
- name: Add dns server entry 
  hosts: localhost
  tasks:
    - lineinfile:
        path: /etc/resolv.conf
        line: 'nameserver 10.1.250.10'
```

The `lineinfile` module is idempotent, that means if we run the playbook on the same host 1000 times, it will add the entry only once. 

### Ansible Plugins

Let's consider you have a complex infrastructure with multiple sites. Every site has virtual machines, load balancers, DBs spread accross different regions. Our goal is to automate the provisioning and configuration of these resources using Ansible.

While ansible provides a rich set of built-in modules and features, we soon realize that we need other functions, for example: an automatic inventory solution that can dynamicly fetch real time information from our cloud (instances, security groups, tags, etc).

But as we know the static inventory file is not enough in managing a dynamic cloud infrastructure. We could also use ansible to dynamicly configurate load balancers, ssl certificates etc. 

To address these challenges you can leverage ansible plugins which provide extensibility and customization options. 
In ansible a plugin is a piece of code that extends or modifies the functionality of ansible. Plugins can be used to enhance various functionalities of ansible (inventories, modules, callbacks). They provide a flexible way to customize our workflow. 
Each plugin type serves a specific purpose and offers unique capabilities for extending ansible functionalities. 

For example if we needed a custom inventory of a proxmox pve cluster we could install this module instead of writing all the logic alone: [Proxmox Plugin](https://docs.ansible.com/ansible/latest/collections/community/general/proxmox_inventory.html)

### Laboratory 7

```yaml
---
- name: 'hosts'
  hosts: all
  become: yes
  tasks:
    - name: 'Execute a script'
      script: '/tmp/install_script.sh'
```

```yaml
---
- name: 'hosts'
  hosts: all
  become: yes
  tasks:
    - name: 'Execute a script'
      script: '/tmp/install_script.sh'
    - name: 'Start httpd service'
      service:
        name: 'httpd'
        state: 'started'
```

```yaml
---
- name: 'hosts'
  hosts: all
  become: yes
  tasks:
    - name: 'Execute a script'
      script: '/tmp/install_script.sh'
    - name: 'Start httpd service'
      service:
        name: 'httpd'
        state: 'started'
    - name: "Create or update index.html file."
      lineinfile:
        path: /var/www/html/index.html
        line: "Welcome to ansible-beginning course"
        create: true
```

```yaml
---
- name: 'hosts'
  hosts: all
  become: yes
  tasks:
    - name: 'Execute a script'
      script: '/tmp/install_script.sh'
    - name: 'Start httpd service'
      service:
        name: 'httpd'
        state: 'started'
    - name: "Update /var/www/html/index.html"
      lineinfile:
        path: /var/www/html/index.html
        line: "Welcome to ansible-beginning course"
        create: true
    - name: 'Create a new user'
      user:
        name: 'web_user'
        uid: 1040
        group: 'developers'
```

### Ansible Handlers

Imagine you are managing a web server infrastructure with multiple servers. You freequently make changes to the web servers configuration to tweak the settings. However, modifying the config file does not apply the changes. To ensure the changes are applied you need to restart the web server service. 
In this scenario, you have to manualy execute a task that restarts the web server service after every configuration update. 
What happens if you forget to do it or something wents wrong?
To ensure everything works properly ansible introduced handlers. With handlers we can define an action to restart the web service and associate it with a task that modifies the configuration file. 
This creates a dependency between the task and the handler. 
Now everytime we modify the config file, an associated handler is triggered. 

Handlers in ansible are special tasks associated with events or notifications. They are tipicly defined in a playbook and executed only when triggered by a task.
Handlers enables you to manage actions that depends on a state or configuration changes on your system. 

```yaml
---
- name: 'Deploy Application'
  hosts: application_1_servers
  tasks:
    - name: Copy code
      copy:
        src: app_code/
        dest: /opt/webapp1
      notify: Restart application service
  
  handlers:
    - name: Restart application service
      service: application_service
      state: restarted
```
If the same handler is requested multiple times in the same playbook it will be executed only once at the end. 

### Ansible Roles

Roles are assigned to servers as jobs are assigned to people. Assigning a role to a server means performing all the tasks that are needed to make a server suitable for the desired role. 
For example do what is needed to make a server a database server. Installing dependencies, database, monitors, alerts etc.

But why we need roles when we could do:

```yaml
---
- name: Prepare a DB server
  hosts: db-server
  tasks:
    - yum:
      - name: pre-req-packages
        state: present
      - name: mysql
        state: present
    - service:
      - name: mysql
        state: started
    - mysql_db:
      - name: db1
        state: present
```

Once it's getting perfectionzied it can be used by hundreds of people that needs to install mysql. 

We can split the above playbook as follows.


```yaml
---
- name: Prepare a DB server
  hosts: db-server
  roles:
    - mysql
```

```yaml
---
  tasks:
    - yum:
      - name: pre-req-packages
        state: present
      - name: mysql
        state: present
    - service:
      - name: mysql
        state: started
    - mysql_db:
      - name: db1
        state: present
```

Thats how we created a Role. The primary scope of roles is to make your work reusable.

Roles introduced a set of best practices that must be followed before creating a role:
- all tasks goes into the `tasks` directory
- all defaults goes into the `defaults` directory
- all vars goes into the `vars` directory
- all handlers goes into the `handlers` directory
- all templates goes into the `templates` directory

Ansible galaxy is a community tool that contains roles almost for everything. 

Using `ansible-galaxy init <rolename>` command will create a structure ready to go for your role.

But how my playbook finds the role i specify? Well we could put in the same folder:

```sh
playbook.yml
roles/rolename
```

Or move the roles in a common directory at the `/etc/ansible/roles` location. This location can be changed in the ansible configuration file. 

After finding a role we can install it using `ansible-galaxy install rolename` or to view the currently installed roles by running `ansible-galaxy list` command. 

### Ansible Collections

Let's consider a scenario. You work as a network engineer, responsible for managing a large network infrastructure. You need to automate and manage the configuration of network devices from various vendors. 
Ansible provides a set of built in modules for network automation. Ansible provides a big collections of ready to go integrations such as `network.cisco`, `network.arista` etc. These are vendor specific modules that provides playbook for managing specific devices.  
By installing these collections you get access to specialized functionalities required to automate the network infrastructure. 
Here is an example command on how to install a collection: `ansible-galaxy collection install network.cisco`, or if you have a requirements file with: `ansible-galaxy collection install -r requirements.yml`.

But what are ansible collections? Ansible collections are a way to package and distribute ansible modules, roles, plugins, etc. A collection is a self-contained unit that encapsulate this components making them easily accessible and shareble. 

Collections can be created by ansible by vendors or everyone else. 

### Laboratory 8

```yaml
---
- hosts: switches
  collections:
    - company_xyz.networking_tools
  tasks:
    - name: Configure VLAN 10
      configure_vlan:
        vlan_id: 10
        vlan_name: Admin_VLAN
```

### Ansible Introduction to templating 

Ansible uses Jinja2 templates. Jinja2 is fully featured templating engine for python. 
Please refer to [Jinja2 official documentation](https://jinja.palletsprojects.com/en/stable/)
We can take variables from our inventories:
```ini
# Web Servers
web1 ansible_host=server1.company.com ansible_connection=ssh ansible_user=root ansible_ssh_pass=Password123!
web2 ansible_host=server2.company.com ansible_connection=ssh ansible_user=root ansible_ssh_pass=Password123!
web3 ansible_host=server3.company.com ansible_connection=ssh ansible_user=root ansible_ssh_pass=Password123!
```

```yaml
---
- name: Update user password
  host: all
  tasks:
    - name: upd user
      user: user pass '{{ ansible_ssh_pass }}'
```

Let's make a more complex example. What if i had a index.html file what has a jinja2 template variable?

```html
<!DOCTYPE html>
<html>
  <body>
    This page is on host: {{ inventory_hostname }}
    </body>
</html>
```

```yaml
---
hosts: web_servers
tasks: 
  - name: Copy index.html to remote servers
    template: 
      src: index.html.j2
      dest: /var/www/nginx/index.html
```

Ansible will create index.html copies before delivering the file to the remote server. 


### Laboratory 9

```yml
---
- hosts: localhost
  connection: local
  vars:
    dialogue: "The name is Bourne, James Bourne!"
  tasks:
    - template:
        src: name.txt.j2
        dest: /tmp/name.txt
```

```yml
{{ dialogue | replace('Bourne', 'Bond') }}
```
-----------------------

```yaml
hostname, ipaddress, monitor_port, type, protocol
{% for host in groups['lamp_app'] %}
{{ host }}, {{ hostvars[host]['ansible_host'] }}, {{ hostvars[host]['monitor_port'] }}, {{ hostvars[host]['protocol'] }}
{% endfor %}
```

```yaml
---
- hosts: monitoring_server
  become: yes
  tasks:
    - template:
        src: agents.conf.j2
        dest: /etc/agents.conf
```
------------------------

```yaml
hostname, architecture, distribution_version, mem_total_mb, processor_cores, processor_count
{% for host in groups['web'] %}
{{ host }}, {{ hostvars[host]['ansible_architecture'] }}, {{ hostvars[host]['ansible_distribution_version'] }}, {{ hostvars[host]['ansible_memtotal_mb'] }}, {{ hostvars[host]['ansible_processor_cores'] }}, {{ hostvars[host]['ansible_processor_count'] }}
{% endfor %}
```

```yaml
- hosts: all
  tasks:
  - setup:

- hosts: localhost
  tasks:
    - template:
        src: inventory.csv.j2
        dest: /tmp/inventory.csv
      run_once: yes
```