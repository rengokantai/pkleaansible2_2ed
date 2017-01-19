# pkleaansible2_2ed
##3. Scaling to Multiple Hosts
### Working with inventory files
defaults to /etc/ansible/hosts. or
```
-i or --inventory-file 
```
option.

####Regular expressions in the inventory file
```
[webserver] 
ke[01:02].fale.io 
```

### Working with variables
#### Host variables
```
[webserver] 
01.fale.io domainname=example1.fale.io 
02.fale.io domainname=example2.fale.io 
```
It will be able to refer to the domain name variable.



[group_vars, inventory](http://docs.ansible.com/ansible/intro_inventory.html)
note, in ansible 2.0
```
ansible_user, ansible_host, and ansible_port
```

or ansible_connection
```
[targets]

localhost              ansible_connection=local
other1.example.com     ansible_connection=ssh        ansible_user=mpdehaan
other2.example.com     ansible_connection=ssh        ansible_user=mdehaan
```
