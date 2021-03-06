# Ansible-Doc
Ansible Documentation - Vardhan notes


====================================
vi etc/sudoers.d

ansible ALL=(ALL) NOPASSWD:ALL

--- generate ssh keys on master =========

ssh-keygen -t rsa 

copy the pub file and pate in .ssh/authorixed keys in client node   before that run the below commands on client node

mkdir -p .ssh
  
    cd .ssh/
   vi authorized_keys
    chmod 700 .ssh/
    cd .ssh/
 chmod 640 authorized_keys
===============================================

to set the idle time

echo 'ClientAliveInterval 50' | sudo tee --append /etc/ssh/sshd_config
/bin/systemctl restart sshd.service


====================================
Inventories  -- two types 

statick and dynamic

-- group 
--variables 


default inventory loaction is /etc/ansible 

alternatively inventory location can be specified through the ansible.cfg file 

ex: 

[webservers]
host1
host2

[dbservers]
host3
host4

[servers:children]  --nested groups
webservers
dbservers


-- you can create your own inventory file --

vi inventory 

ansible -i inventory all --list-all

===================ad hoc commands ==================== ----

ansible all -m ping 

ansible all -m shell -a "df -h"   -- to list the directories

     -a we need to use before the command 

ansible all -m yum -a "name=htttpd state=present"  -- to install the package 

ansible all -m yum -a "name=httpd state=removed"    -- to remove the package 

to install package -- present

to remove the package -- removed

-- to start the service 

 ansible all -m service -a "name=http state=stssarted"

service start --started
service stopo -- stopped
service restart -- restarted


-------copy module --
to copy a file 

ansible all -m copy -a "src=/root/divyansh dest=/root/"

==============================================================================================
user is the module to create 

to create the user 

 ansible all -m user -a "name=divyansh password=divyansh" 

or  ansible all -m user -a "name=divyansh state=present"

to remove

 ansible all -m user -a "name=divyansh state=absent"

====================================================

How to create a directory 

 ansible all -m file -a "name=sample state=directory mode=777" 

    here mode we are using it for setting up the permissions 

to delte the directory 

ansible all -m file -a "name=sample state=absent"


======================================================
to create a file 

ansible all -m shell -a "name=subuli state=file mode=777"


to list the available modules -- ansible-doc -l

if you want to know more about module --ansible-doc user


=== To check playbook syntax === ansible-playbook --syntax-check <yamlfilename>


 
ansible-playbook -C <yaml file>   dry run -- use -C option 

if you want to know more details of the output use -vvvv option at the end


==========================

remote_user: the name of the remote user 
become: to enable or disable the privilage escalation
become_method: to allow using an alternative escalation solution 
become_user: the target user used for privilege escalation 


====================variables in ansible==================

you can define variables in playbook -- but this is not recomendded, as playbook should be static and reusable

instead of defining varible in playbook , we can define varibale in below
inventory  (deprecated)
inclusion files
local fact



variable precedence --

Varibles scope:

Global Scope: this is when a variable is set from inventory or the command line 

Play Scope: this is applied when it is set from a play
Host scope: this is applied when set in inverntory or using a host variable inclusion file

--> when the same variable is set at different levels, the most specific level gets precedence
     host
     playbook
     global scope

---> when a variable is set from the command line, it will overwrite anything else 

    ex: ansible-playbook site.yaml -e "web_package=apache"


---> inventory at hostlevl  variables define

  [web_servers]

web_server_1 ansible_user=centos http_port=80
web_server_2 ansible_user=ubuntu http_port=8080

at hostgroup level:

 [web_servers:vars]
ansible_user=centos
http_port=80



Register a variable: 

Ansible register variable or ansible register module is used to capture or store the output of the command or task. By using the register module, you can store that output into any variable.


array as variable syantax:

- hosts: all
  vars:
    <variablename>:
    - <arrayvalue1>
    - <arrayvalu2>
    - <arrayvalue3>


Disctionary as variable:
  - hosts: all
    vars:
      <variablename>:
       <key>: <value>

=======ansible vault==========================================

Ansible Vault is a feature that allows users to encrypt values and data structures within Ansible projects. 

commands:

create: to create ansible vault file in the encrypted format
view: to view data of encrypted file
edit: to edit encrypted file
encrypt: to encrypt an unencrypted file (meaning if you want to encrypt your existing file)
decrypt : to decrypt an encrpted file

--ask-vault-pass : to provide password while running playbook

--vault-password-file: to pass a vault password through a file



example:  here vault.yaml is the file name

  1) ansible-vault create vault.yaml
 
  2) ansible-vault view vault.yaml 
  3) ansible-vault edit vault.yaml 
  
  4)  ansible-vault  decrypt vault.yaml 

  5) ansible-vault encrypt vault.yaml
 


================================

==================Ansible facts:============================

Notes:

---> Ansible facts are variables that are automatically set and discovered by ansible on managed hosts
----> facts contain information about hosts that can be used in conditionals
---> for instance, before installing specific software you can check that a managed host runa specific kernmel version
---->By default , all playbooks perform fact gathering before running the actual plays
---> you can run facts gathering in an adhoc command using the setup module
---> to show facts, use debug module to proint the value of the ansible_facts variable

Ansible Gathered Facts or playbook variables belongs to one of the following types

Dictionary
List
AnsibleUnsafeText


How to know the data type of variable (or) fact:  {{ <the variable name> | type_debug }}


custome facts:

steps to create custome facts:

1) create /etc/ansible/facts.d on remote machine
2) Inside of facts.d place one or more custome facts files (facts file extension should be *.fact)
3) the output of *.fact file shoud be json
4) the *.fact file should have execution permission 


#!/bin/bash
git_ver=$(git --version | awk '{ print $3 }')
echo "{\"Git_Version\":\"${git_ver}\"}"

example palybook is here: ansible_locak is the default variuable that we can use to fetch the custom facts

- hosts: all
  tasks:
  - name: fetching custom facts from remote machine
    debug:
     msg: "the cutome fact is {{ansible_local}}"


                                                                                                                                                                                                       
--------------
Conditionals
Loops
Roles
User mailing list
filters-- Jinja2 filters and their templates
                                  
====conditions===

when:

syntax
tasks:
modelue: your code
when: <expression>


if your expression is true then will execute the module othere wise it will not execute

Register: Register key is used to store the response to a vaiable after completion of each task
 we have to use at the end of each task as follows


---delegate to  -- the below task execute only on local host even if you are using hots:all in header

---
 - name: name of the playbook
   hosts: all
   tasks:
   - command:
      touch: /tmp/vardhan.txt
     delegate_to: localhost


when can be used to test multiple conditions as well
use and or or and group the conditions with parentheses

ex: when: ansible_distribution == "Centos" or(and) 
           ansibel_distribution == "RedHat"

the when keyword also supports a list , and when using a list, all of the conditions must be true
complex conditional statements can group conditions using parentheses


===============handlers============

--> handlers allow you to configure playbooks in a way that one taks will only run if another task has been running successfully
--> in order to run the handler, a notify statement is used from the main task to trigger the handler

--> handlers typically are used to restart services or reboot hosts
--->handlers are executed after running all tasks in a play
---> handlers will only run if a task has changed something, so if an ok result instead of changed result is reported , the handler will not run 
--> if one of the task fails , the handler will not run, but this may be oberwritten using force_handlers:True

--> one task may trigger more than one handler


===========Blocks==========

--> A block is a logical group of tasks
--> it can be used to control how tasks are executed
--> one block can, for instance, be enabled using a sinle "when"
--> Blocks can also be used in error condition handling 

    use "block" to define the main tasks to run 
    use "rescue" to define tasks that run if tasks defined in the block fail
    use "always" to define tasks that wull run , regardless of the success or failure of the "block" and "rescue" tasks

--> Notice that items cannot be used on blocks


==========Understanding failure handling==

--> Ansible looks at the exit status of task to determine whether it has failed
--> When any task fails, ansible aborts the rest of the play on that host and continues  with the nexy host
--> Diferent Soluctions can be used to change that behaviour
--> use "ignore_errors" in task to ignore failures
--> use "force_handlers" to force a handler that has been triggered to run, even if(another) task fails

== Defining Failure States===

--> As ansible only looks at the exit status of a failed task, it may think a task was successfull where that is not the case
--> to be more specific., use "failed_when" to specify what to look for in command output to recognize a failure

====using the fail module============

--> The "failed_when" keyword can be used in task to identify when a task has failed
--> The "fail" module can be used to print a message that informs why a task has failed
--> To use "failed_when" or "fail" , the result of the command must be registered, and the registered variable output must be analyzed
--> When using the "fail" module, the failing task must have "ignore_errors" set to yes


========loop========== loop=with_*

The loop keyword is the current keyword replaced by with_items

The synatax will be probably be deprecated in future versions of ansible
  with_items: equivalant to loop keyword
  with_file: the item conatins a file, which contents is used to loop through
  with_sequence: generates a list of values based on numaric sequence


