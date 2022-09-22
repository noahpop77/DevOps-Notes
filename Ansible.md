# Ansible Notes

Ansible is an automation software used to manage many servers
Ansible official documentation: https://docs.ansible.com

# Inventory File
Inventory files are files used by Ansible as a target for the commands that are to be executed. When it comes to the formatting for inventory files, there are a bunch of options. Below are examples and lists of options, as well as their explanations. 

Formatting:
```
web ansible_host=<ip address> ansible_connection=ssh ansible_user=root
target1 ansible_host=<hostname> ansible_connection=winrm ansible_user=admin

[home-network]
mail ansible_host=<ip address> ansible_connection=ssh ansible_sssh_pass=mypassword
http ansible_host=<ip address> ansible_connection=ssh
```

`ansible_connection=ssh`
`ansible_connection=winrm`
The ansible_connection option is used for specifying what type of connection will be used for the management commands. For certain Windows machines, there is the `winrm` connection type which manages windows remote connections. For Linux and most other operating systems we use the `ssh` connection type.

`ansible_user`
It is used to specify which user is going to be connecting to the machine.

`ansible_sssh_pass`
It is used to specify a password used to connect to certain machines. Under normal circumstances we should not use this and use ssh keys instead.

`ansible_host`
This field specifies the address of the target. If DNS is configured properly you can add a hostname in here, otherwise an IP addresses works just fine.

`ansible_port`
Used to specify which port will be used for the specific target

`Target Alias`
The very first entry in each line is the alias that is used for the target. That way during the ansible commands we can specify a certain host or a group of them

`[name]`
This is used for specifying a group that we can target with ansible commands.


# Ansible Playbooks

All playbooks are written in YAML format. A playbook is a single YAML file that contains a list of plays/tasks. Examples of tasks are executing commands on the script, or issuing any other instructions on the machine.

This is an example playbook that shows most of the functions a playbook can perform, but there are MANY MORE.

Each first `-` is a new command/task initiative on a specific host. Everything under the task section for that entry is a list of commands that will be run on it
```
-
    name: 'Stop the web services on web server nodes'
    hosts: web_nodes
    tasks:
        -
            name: 'Stop the web services on web server nodes'
            command: 'service httpd stop'
        -
            name: 'status of the web services on web server nodes'
            command: 'service httpd status'
-
    name: 'Shutdown the database services on db server nodes'
    hosts: db_nodes
    tasks:
        -
            name: 'Shutdown the database services on db server nodes'
            command: 'service mysql stop'
-
    name: 'Restart all servers (web and db) at once'
    hosts: all_nodes
    tasks:
        -
            name: 'Restart all servers (web and db) at once'
            command: '/sbin/shutdown -r'
-
    name: 'Start the database services on db server nodes'
    hosts: db_nodes
    tasks:
        -
            name: 'Start the database services on db server nodes'
            command: 'service mysql start'
-
    name: 'Start the web services on web server nodes'
    hosts: web_nodes
    tasks:
        -
            name: 'Start the web services on web server nodes'
            command: 'service httpd start'

```

For the `hosts` section of the playbook you can specify a host name, or you can specify a pattern. Some patterns could look like:
![[Pasted image 20220905023342.png]]

- When it comes to specifying an inventory for running a playbook you can specify a playbook like this:
`ansible-playbook -i inventory.txt playbook.yml`
- you can also run:
`ansible-playbook -i inventory.py playbook.yml`
The python script is responsible for reaching out to sources you have in the inventory and giving Ansible back a list of groups, hosts and their information. There are many scripts available for this on the ansible documentation site.


# Ansible Modules
Modules can be run in a playbook or ad-hoc commands. To run an ad-hoc command, you will need to do the command below. This example runs the ping module against all hosts listed under the `all` group in the inventory file within the current directory.
`ansible all -m ping -i inventory`

The other way is to run playbooks, which are a list of commands in a sequence written in YAML format. To run a playbook you can run the following command. This just runs the playbook named `myplaybook.yml`
`ansible-playbook myplaybook.yml`

Modules are Ansible programs that are made to execute on remote machines. There are a plethora of different modules, all for different purposes and environments. Some of which are modules for:
```
- System
- Commands
- Files
- Database
- Cloud
- Windows
- More....
```

#### Some notable modules are:
`command`
Runs a command on the remote system and takes in free form input, meaning you can write anything on that line, and it will run the whole line.

`script`
This command takes a script on your master and copies it to all the slaves, and then executes it. 

Here is an example of this module in action
```
-
	name: Play 1
	hosts: localhost
	tasks:
		-
			name: Run a script on remote server
			script: /path/to/script -arg1 -arg2
```

`service`
This module is used to dictate the state of a service on the remote machine. You can have the state in 3 states, Started, Stopped, and Restarted. The service module is not a free form input type of module and requires a specific key value pair, in this case it is a service name and a state. In this example, we are showing a PostgreSQL database being started.
```
-
	name: Play 1
	hosts: localhost
	tasks:
		-
			name: Starrt the database service
			service: name=postgresql state=started

OR IT CAN BE WRITTEN AS THIS AS WELL

-
	name: Play 1
	hosts: localhost
	tasks:
		-
			name: Starrt the database service
			service:
				name: postgresql
				state: started
```

The reason that it is written as `state: started` and not `state: start` is because of the way Ansible handles these things. We want the service to be turned on and in this case because of idempotency this ensures that if the service is off it will start it and if it is already on it will do nothing. It is a target state rather than a command.

`lineinfile`
The lineinfile module is used to find a line in a file and replace it or add it if it doesn't exist. For example, if we want to add a DNS server to the `/etc/resolv.conf` file we can run the following playbook.
```
-
	name: Add DNS server to resolv.conf
	hosts: localhost
	tasks:
		- lineinfile:
			path: /etc/rresolv.conf
			line: 'nameserver 10.1.12.12'
```
The lineinfile module is idempotent. This module will ensure that there is only a single entry in the file and not many copies for no reason.

#### Custom Modules
You can also develop custom modules. There is more information on that here.
https://docs.ansible.com/ansible/latest/dev_guide/index.html

# Variables

format is jinja2 templating

```
-
	name: Add DNS server to resolv.conf
	hosts: localhost
	vars:
		dns_server: 10.12.12.12
	tasks:
		- lineinfile:
			path: /etc/rresolv.conf
			line: 'nameserver {{ dns_server }}'
```

```
# Sample inventory file
web http_port=8081 snmp_port=161-162 inter_ip_range=192.0.2.0
```

```
# Sample variable file - web.yml
http_port: 8081 
snmp_port: 161-162 
inter_ip_range: 192.0.2.0
```

```
-
	name: Set Firewall Configurations
	hosts: web
	tasks:
		- firewalld:
			service: https
			permanent: true
			state: enabled
			
		- firewalld:
			port: '{{ http_port }}'/tcp
			permanent: true
			state: enabled
			
		- firewalld:
			port: '{{ snmp_port }}'/udp
			permanent: true
			state: enabled

		- firewalld:
			source: '{{ inter_ip_range }}'/24
			permanent: true
			state: enabled
```

![[Pasted image 20220903110926.png]]

# Conditionals
You can assign conditionals based off certain criteria in your playbooks


This example conditional shows has an `if` conditional to run if the certain os version/family is present.

```
-
	name: Install NGINX
	hosts: all
	tasks:
		- name: Install NGINX on Debian
		  apt: 
			name: nginx
			state: present
		  when: ansible_os_family == "Debian" and ansible_distribution_version == "16.04"

		- name: Install NGINX on Debian
		  yum: 
			name: nginx
			state: present
		  when: ansible_os_family == "RedHat" or ansible_os_family - "SUSE"

```


Loops plus conditional
```
-
	name: Install NGINX
	hosts: all
	vars:
		packages:
			- name: nginx
			  required: True
			- name: mysql
			  required: True
			- name: apache
			  required: false

	tasks:
		- name: Install "{{ item.name }}" on debian
		  apt: 
		    name: "{{ item.name }}"
			state: present
		  when: item.required == True         # add conditional to the loop
		  loop: "{{ packages }}"              # loops through packages

```

Conditionals and Registers
Takes the output of the status command checking httpd and saves it to a register named result which is inspected for the word down to see if the service is down and then it sends an email if the service is down
```
-
	name: Check status of a service and email if its down
	hosts: localhost
	tasks:
		- command: service httpd status
		  register: result

		- mail:
		    to: admin@company.com
			subject: Service Alert
			body: HTTPD service down
			when: result.stdout.find('down') != -1
```

# Loops

Looping function of Ansible playbooks.

```
-
	name: Create Users
	hosts: localhost
	tasks:
		- user: name='{{ item }}'     state=present
		  loop:
			  - joe
			  - george
			  - ravi
			  - mani
			  - bob
			  - dylan
			  - joe
			  - staniago
			  - timmy



PASSING 2 VALUES TO LOOP INSTEAD OF 1 STRING


-
	name: Create Users
	hosts: localhost
	tasks:
		- user: name='{{ item.name }}'     state=present    uid='{{ item.uid }}' 
		  loop:
			  - name: joe
			    uid: 1
			  - name: george
			    uid: 2
			  - name: ravi
			    uid: 3
			- name: mani
			    uid: 4
			- name: bob
			    uid: 5
			- name: dylan
			    uid: 6
			- name: joe
			    uid: 7
```

There are also specialized different loop directives available instead of just the loop function. The loop function will iterate through all given data points provided, whereas the `with_whatever` directive iterates over more specific things.

![[Pasted image 20220904164456.png]]
![[Pasted image 20220904164554.png]]
![[Pasted image 20220904164559.png]]


# Roles

Assigning roles to servers.
Normal functions of roles are to define tasks used for a certain type of service like:
- Installing prerequisites
- installing packages
- configuring services
- configuring users and such

You can do this with a playbook, but since so many people need to run code like this, it made sense for this to be packaged. Roles are also a set of best practices. Roles also help with sharing code with Ansible community. https://galaxy.ansible.com/ is like dockerhub for Ansible roles and playbooks. An example of this could look like

```
- name: Install and configurre MySQL
  hosts: db-server1
  roles:
	  - mysql
```

The other version of this code would look like

```
- name: Install and configurre MySQL
  hosts: db-server1
  tasks:
	  - name: install pre-requisites
	    yum: name=pre-req-packages state=present
	  - name: install MySQL packages
	    yum: name=mysql state=present
	  - name: startMySQL service
	    service: name=mysql state=started
	  - name: Configure Database
	    mysql_db: name=db1 state=prresent
```

There is an `ansible-galaxy` package/command that can be used to generate a skeleton directory structure like the one shown below to begin working with making your own roles. One way you can include your own roles in a project or playbook is to make a roles directory in the same project directory as your playbook then include the role in that directory or make a roles file on your machine for all playbooks to use at `/etc/ansible/roles` and reference it with something like `roles_path = /etc/ansible/roles`

![[Pasted image 20220905022011.png]]

```
- 
  name: Install and configurre MySQL
  hosts: db-server1
  roles:
    -  geerlingguy.mysql
    - nginx


ORRRRRRRRRRRRRRRRR

- 
  name: Install and configurre MySQL
  hosts: db-server1
  roles:
    -  geerlingguy.mysql

- 
  name: Install and configurre web server
  hosts: web-server
  roles:
    -  nginx
```
