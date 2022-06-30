Ansible Inventory:-
An Ansible controller needs a list of hosts and groups of hosts, where commands, modules, and tasks are performed on the managed nodes, this list is known as inventory.
-  It may contain information such as – Host IPs, DNS Name, SSH User and Pass, SSH Port service info (in case it is other than port 22). 
- default inventory is hosts.yaml

Q) Creating custom inventory file(inventory1.txt)?
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

#Ansible Role
- no need to include tasks in main,yml - do practise 
- ansible galaxy init ansible_role1
- remove direcotry/everything inside $rm -rvf *


10. Asynchronous Actions and Polling
- A tasks takes 10 minutes to execute on slave1, now let's consider SSH connection broken?
- Ansible runs tasks synchronously by default. It keeps the connection open to the remote node until the task is completed.
-Ansible provides you Asynchronous mode, which lets you control how these long-running tasks execute. We can tell Ansible to check the status of the task after any particular interval.


- 
  name: this is our 1st play.
  hosts: webserver1
  tasks: 
    - 
      name: 'sleep for 60 sec'
      command: sleep 60
      async: 70
      poll: 35
    - 
      name: 'second task'
      command: touch /tmp/second_task.txt
	  
Setting poll equal to 0 (FIRE-AND-FORGET)


11.Strategies in ansible
# By default, plays run with a linear strategy, in which all hosts(host1,host2) will run each task before any host starts the next task.
# control waits until all hosts complete task1 then it will go to next task
# problem - node1 completes task in 10s and nod2 takes more time, So meanwhile node1 sits ideal. 

- 
  name: 'strategy demo'
  hosts: webserver1,sqlserver1
  strategy: linear # strategy: free
  tasks:
    - 
      name: 'first task'
      apt: name='apache2' state='present'
    - 
      name: 'second task'
      command: touch /tmp/strategy_task2.txt
	  
#free Strategies-
- if you don't want dependency 


Q) suppose we have 100 hosts so what is the default value on which run starts?
config file -> fork =5 (default value) - make sure master system configuration is good in case you increase fork value
# or change in playbook - here host1 will complete all tasks then move to host2

- name: 'strategy demo'
  hosts: webserver1,sqlserver1
  serial: 1
   tasks:
    - name: 'first task'
      apt: name='apache2' state='present'
    - name: 'second task'
      command: touch /tmp/strategy_task2.txt
    - name: '3rd task'
      command: sleep 30
	  
12. error handling -ignore_errors: True
# if any error comes on host1 then next task doesn't start on host1, But on host2 both tasks will execute 

- 
  name: 'strategy demo'
  hosts: webserver1,sqlserver1
  tasks:
    - 
      name: 'first task'
      command: touch /tmp/task/task1.txt  # if any error comes then next task doesn't execute use below line
	  ignore_errors: True
    - 
      name: 'second task'
      command: touch /tmp/task2.txt
	  
13. Jinja 2
# Ansible uses Jinja2 templating to enable dynamic expressions, dynamic file generation based on its parameter, and access to variables. 
# print variable in upper,lower,replace list- min,max,unique,random like anything
- tasks: 
    - 
      debug: 
        msg: "Hello {{ your_name  }}"
    - 
      debug:
        msg: "Hello {{ your_name | upper  }}"
		
14. ansible vault
#in inventory file like hosts file has username, password but want to encrypt 
#To encrypt it using Ansible Vault
$ansible-vault encrypt inventory.txt --output enc_inven.txt
#Viewing encrypted file
ansible-vault view enc_inven.
#Running Ansible with Vault-Encrypted Files
ansible-playbook -i enc_inven.txt playbook.yml --ask-vault-pass

15. ansible lookups
#a huge amount of configuration data or server details being stored in the different sources having formats like .csv, txt or INI, etc.
# lookup plugins, we can query data from these external resources. These sources can be local files or some external datastores. 
#ansible_ssh_pass: "{{ lookup('<file_name>', '<path_of_lookup_file>') }}"

-
  name: Test Connectivity
  hosts: webserver1
  vars:
     ansible_ssh_pass: "{{ lookup('csvfile', 'webserver1 file=credentials.csv delimiter=,') }}"
	 #ansible_ssh_pass: "{{ lookup('ini', 'password section=webserver1 file=credentials.ini') }}"
  tasks:
  - name: create a dummy file on webserver
    command: touch /tmp/csv_lookups.txt
	
16. dynamic inventory in ansible
Mostly, any infrastructure can be managed with a custom inventory file, but there are many situations where more control is needed.
Consider you are working with the 100 machines on AWS cloud. Now suppose you have to restart all these machines
which will lead to change in its public addresses and hence we have to make changes in the host file as well.
This is where Dynamic inventory comes very handy.

#practise this

17.custom module in ansible

#practise this
https://www.youtube.com/watch?v=ez4ihEtauxs&list=PL6XT0grm_TfgP3OlZzmGh4Cq_rHtX8z7e&index=18
18. Ansible tutorial: control a windows machine from ansible
