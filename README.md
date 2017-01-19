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
```
[webserver] 
01.fale.io domainname=example1.fale.io 
02.fale.io domainname=example2.fale.io 
```
It will be able to refer to the domain name variable.
