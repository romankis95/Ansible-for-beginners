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

```
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

