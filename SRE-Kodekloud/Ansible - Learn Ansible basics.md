

## Inventory file

Example of inventory file

inventory 
```ini
# Sample Inventory File

# Web Servers
web1 ansible_host=server1.company.com ansible_connection=ssh ansible_user=root ansible_ssh_pass=Password123!
web2 ansible_host=server2.company.com ansible_connection=ssh ansible_user=root ansible_ssh_pass=Password123!
web3 ansible_host=server3.company.com ansible_connection=ssh ansible_user=root ansible_ssh_pass=Password123!
db1  ansible_host=server4.company.com ansible_connection=winrm ansible_user=administrator ansible_password=Dbp@ss123!
```



Another inventory file example with groups

```ini
# Web Servers
[web_servers]
web1 ansible_host=server1.company.com ansible_connection=ssh ansible_user=root ansible_ssh_pass=Password123!
web2 ansible_host=server2.company.com ansible_connection=ssh ansible_user=root ansible_ssh_pass=Password123!
web3 ansible_host=server3.company.com ansible_connection=ssh ansible_user=root ansible_ssh_pass=Password123!

# Database Servers
[db_servers]
db1 ansible_host=server4.company.com ansible_connection=winrm ansible_user=administrator ansible_password=Password123!


[web_servers]
web1
web2
web3
```



Lets now create an inventory with parent group called all_servers and within this group lets add web_servers and db_servers.

```ini
[all_servers:children]
web_servers
db_servers

# Web Servers
web1 ansible_host=server1.company.com ansible_connection=ssh ansible_user=root ansible_ssh_pass=Password123!
web2 ansible_host=server2.company.com ansible_connection=ssh ansible_user=root ansible_ssh_pass=Password123!
web3 ansible_host=server3.company.com ansible_connection=ssh ansible_user=root ansible_ssh_pass=Password123!

# Database Servers
db1 ansible_host=server4.company.com ansible_connection=winrm ansible_user=administrator ansible_password=Password123!


[web_servers]
web1
web2
web3

[db_servers]
db1
```


Lets create an inventory file with web_nodes, db_nodes, boston_nodes, dallas_nodes and us_nodes within boston_nodes and dallas_nodes

```ini
# Web Servers
[web_nodes]
web_node1 ansible_host=web01.xyz.com ansible_connection=winrm ansible_user=administrator ansible_password=Win$Pass
web_node2 ansible_host=web02.xyz.com ansible_connection=winrm ansible_user=administrator ansible_password=Win$Pass
web_node3 ansible_host=web03.xyz.com ansible_connection=winrm ansible_user=administrator ansible_password=Win$Pass

# DB Servers
[db_nodes]
sql_db1 ansible_host=sql01.xyz.com ansible_connection=ssh ansible_user=root ansible_ssh_pass=Lin$Pass
sql_db2 ansible_host=sql02.xyz.com ansible_connection=ssh ansible_user=root ansible_ssh_pass=Lin$Pass

[boston_nodes]
sql_db1
web_node1

[dallas_nodes]
sql_db2
web_node2
web_node3

[us_nodes:children]
boston_nodes
dallas_nodes
```


## Tasks output with debug module 

```yaml
...
tasks:
  - shell: cat /etc/hosts
    register: result
  - debug: 
    var: result
```

Each module may present different output. 
We can also display a specific part of the output, exmaple:

```yaml
var: result.stdout
# OR 
var: result.rc # to display return code.
```


Another way to display more information while running a playbook without using the debug module is to use -v in the CLI. Example:

```sh
ansible-playbook -i inventory playbook.yaml -v
```


### Magic variables

We can get variables from another host also

```yaml
tasks:
- debug: 
    msg: '{{ hostvars['web2'].dns_server }}'
```



## Ansible facts, gathering facts


During a Ansible playbook execution before running any task, Ansbible run an automatic Facts which collect all the details from the host, information like, OS version, kernel, processor, memory, architecture, volumes mounted, network configuration, disk spaces, etc...

```yaml
# gathering_facts: no # disable playbook from gathering facts
tasks:
- debug:
    var: ansible_facts
```