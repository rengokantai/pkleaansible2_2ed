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

### Working with dynamic inventory

### Amazon Web Services
Download ec2.py , ec2.ini.
```
ansible -i ec2.py all -m ping
```
### DigitalOcean
Download digital_ocean.ini and digital_ocean.py
```
ansible -i digital_ocean.py all -m ping
```

### Working with iterates in Ansible
example
```
--- 
- hosts: webserver 
  remote_user: ansible 
  tasks: 
  - name: Ensure the HTTPd package is installed 
    yum: 
      name: httpd 
      state: present 
    become: True 
  - name: Ensure the HTTPd service is enabled and running 
    service: 
      name: httpd 
      state: started 
      enabled: True 
    become: True 
  - name: Ensure HTTP can pass the firewall 
    firewalld: 
      service: http 
      state: enabled 
      permanent: True 
      immediate: True 
    become: True 
  - name: Ensure HTTPS can pass the firewall 
    firewalld: 
      service: https 
      state: enabled 
      permanent: True 
      immediate: True 
    become: True 
```
#### Standard iteration - with_items
combine latest two items
```
- name: Ensure HTTP and HTTPS can pass the firewall 
    firewalld: 
      service: '{{ item }}' 
      state: enabled 
      permanent: True 
      immediate: True 
    become: True 
    with_items: 
    - http 
    - https 
```
