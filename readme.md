Ansible Inventory:-
An Ansible controller needs a list of hosts and groups of hosts, where commands, modules, and tasks are performed on the managed nodes, this list is known as inventory.
-  It may contain information such as – Host IPs, DNS Name, SSH User and Pass, SSH Port service info (in case it is other than port 22). 
- default inventory is hosts.yaml

Q) Creating custom inventory file?
- ansible.cfg file -> Uncomment  host_key_checking = False
- create inventory.txt
webserver ansible_host=192.168.25.15 ansible_ssh_pass=password ansible_connection=ssh ansible_port=22 ansible_user=root 
sqlserver ansible_host=192.168.25.16 ansible_ssh_pass=password ansible_connection=ssh ansible_port=22 ansible_user=root
-  Executing custom inventory file (-m means modules)
$ansible webserver -m ping -i inventory.txt 
$ansible sqlserver -m ping -i inventory.txt 
$ansible all -m ping -i inventory1.txt

Ansible Playbooks:- 
- list of tasks/instructions to perform on the machines listed in the inventory file
- Playbooks offer repeatable, reusable, simple configuration management.
- Different YAML Tags name, hosts, vars, tasks

- 
  name: "this is our first play."
  hosts: webserver1
  tasks:
    - 
      name: "creating dummy file on webserver1"
      command: touch /tmp/file_on_webserver1
    - 
      name: "copy hosts files in tmp folder"
      command: cp /etc/hosts /tmp/myhosts  #copy: src=test.yml dest=/tmp/
- 
  name: "this is our second play."
  hosts: web_database_servers
  tasks:
    - 
      name: "creating directory in tmp direcotry"
      command: mkdir /tmp/mySecondPlayDir
    - 
      name: "create a dummy file in database and webserver."
      command:  touch /tmp/mySecondPlayDir/secondPlay.txt

	  
Ansible Modules:-
- Modules are individual units of code that can be used from the command line or in a playbook task.
- Ansible executes the modules on the remote managed nodes and captures the return values.
- example  copy, service, ping, command, etc. All modules return JSON format data.

You can execute modules from the command line:
ansible webservers -m service -a "name=jenkins state=started"
ansible webservers -m ping
ansible webservers -m command -a "/sbin/reboot -t now"

#Lineinfile module:-
- used to insert a line, modify, remove, and replace an existing line.

- 
  name: this is our first play.
  hosts: webserver1
  tasks: 
    - 
      name: "create a dummy file on websever1"
      lineinfile: path=/tmp/test.txt line="gaurav"
	  
#Script module:-
script module runs a local script on a remote server after transferring it.

- 
  name: this is our first play.
  hosts: webserver1
  tasks: 
    - 
      name: "create a dummy file on websever1"
      script: testScript.sh Gaura
	  
#Service module
Service module controls services on remote hosts. It can be used to start, stop or check the status of the service on the remote server. 
- state: started #stopped


Idempotency:-
- Ansible remembers its state 

#Package management module
There is a module for most popular package managers, such as DNF and APT

- name: install the latest version of Apache and MariaDB
  dnf: name:httpd state: latest
  apt: name=nginx state=latest


#Variable

- 	name: usign variable in playbook
	hosts: webserver1
	vars:service
	name: vsftpd
	tasks:
	  - name: 'creating file using variable'service: 
		name={{ servicename }} state=stopped
		
#Modularization
In any scripting language, we organize our code using classes, methods, variables, etc. we can modularize the Ansible playbooks to use them efficiently. We can store the repeated tasks or variables in some other file and reference them in our main playbook. 
- create a  tasks.yml file and keep all task here and variable.yml for variables
- references the variable.yml and tasks.yml file in main.yml

#variable.yml 
var1: var1
var2: var2
var3: var3
var4: var4
var5: var5

#tasks.yml
- name: 'task 1'
  command: touch /tmp/task/{{ var1 }}.txt
- name: 'task 2'
  command: touch /tmp/task/{{ var2 }}.txt
- name: 'task 3'
  command: touch /tmp/task/{{ var3 }}.txt
- name: 'task 4'
  command: touch /tmp/task/{{ var4 }}.txt
- name: 'task 5'
  command: touch /tmp/task/{{ var5 }}.txt

#Include
- name: this is our first play.
  hosts: webserver1
  vars_files: 
      - variable.yml
  tasks: 
    - include: task.yml   #used for task include

