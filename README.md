# pkleaansible2_2ed
some review:
```
command shell notify handlers register debug template when
```
for yum and apt, use k-v instead of inline style.(2.0+)  
YES:
```
apt:
  name: apache2
  state: latest
```
NO:
```
apt: name=apache2 state=latest
```
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

#### Nested loops - with_nested
Cartesian product
```
--- 
- hosts: all 
  remote_user: ansible 
  vars: 
    users: 
    - alice 
    - bob 
    folders: 
    - mail 
    - public_html 
  tasks: 
  - name: Ensure the users exist 
    user: 
      name: '{{ item }}' 
    become: True 
    with_items: 
    - '{{ users }}' 
  - name: Ensure the folders exist 
    file: 
      path: '/home/{{ item.0 }}/{{ item.1 }}' 
      state: directory 
    become: True 
    with_nested: 
    - '{{ users }}' 
    - '{{ folders }}' 
```

#### Fileglobs loop - with_fileglobs
```
--- 
- hosts: all 
  remote_user: ansible 
  tasks: 
  - name: Ensure the folder /tmp/iproute2 is present 
    file: 
      dest: '/tmp/iproute2' 
      state: directory 
    become: True 
  - name: Copy files that start with rt to the tmp folder 
    copy: 
      src: '{{ item }}' 
      dest: '/tmp/iproute2' 
      remote_src: True 
      # remote_src default=false, true means copy remote to remote.
    become: True 
    with_fileglob: 
    - '/etc/iproute2/rt_*' 
```


#### Integer loop - with_sequence
```
--- 
- hosts: all 
  remote_user: ansible 
  tasks: 
  - name: Create the folders /tmp/dirXY with XY from 1 to 10 
    file: 
      dest: '/tmp/dir{{ item }}' 
      state: directory 
    with_sequence: start=1 end=10 
    become: True 
```
syntax : with_sequence: start=1 end=10 



##4.  Handling Complex Deployment
### Working with the local_action feature
```
- hosts: database 
      remote_user: ansible 
      tasks: 
      - name: Count processes running on the remote system 
        shell: ps | wc -l 
        # remote command
        register: remote_processes_number 
      - name: Print remote running processes 
        debug: 
          msg: '{{ remote_processes_number.stdout }}' 
      - name: Count processes running on the local system 
        local_action: shell ps | wc -l 
        # local command
        register: local_processes_number 
      - name: Print local running processes 
        debug: 
          msg: '{{ local_processes_number.stdout }}' 
```
note msg: register.stdout

### Delegating a task
delegate a task to local system
```
--- 
    - hosts: database 
      remote_user: ansible 
      tasks: 
      - name: Count processes running on the remote system 
        shell: ps | wc -l 
        register: remote_processes_number 
      - name: Print remote running processes 
        debug: 
          msg: '{{ remote_processes_number.stdout }}' 
      - name: Count processes running on the local system 
        shell: ps | wc -l 
        delegate_to: localhost 
        register: local_processes_number 
      - name: Print local running processes 
        debug: 
          msg: '{{ local_processes_number.stdout }}' 
```

### Working with conditionals
```
--- 
    - hosts: webserver 
      remote_user: ansible 
      tasks: 
      - name: Print the ansible_os_family value 
        debug: 
          msg: '{{ ansible_os_family }}' 
      - name: Ensure the httpd package is updated 
        yum: 
          name: httpd 
          state: latest 
        become: True 
        when: ansible_os_family == 'RedHat' 
      - name: Ensure the apache2 package is updated 
        apt: 
          name: apache2 
          state: latest 
        become: True 
        when: ansible_os_family == 'Debian' 
```
#### Boolean conditionals
```
--- 
- hosts: all 
  remote_user: ansible 
  vars: 
    backup: True 
  tasks: 
  - name: Copy the crontab in tmp if the backup variable is true 
    copy: 
      src: /etc/crontab 
      dest: /tmp/crontab 
      remote_src: True 
    when: backup # vars: backup: True
```
######use command line to pass extra vars to override
```
ansible-playbook -i hosts crontab_backup.yaml --extra-vars="backup=False"
```



#### Checking if a variable is set
```
--- 
    - hosts: all 
      remote_user: ansible 
      vars: 
        backup: True 
      tasks: 
      - name: Check if the backup_folder is set 
        fail: 
          msg: 'The backup_folder needs to be set' 
        when: backup_folder is not defined    # momorize this.  xxx is not defined
      - name: Copy the crontab in tmp if the backup variable is true 
        copy: 
          src: /etc/crontab 
          dest: '{{ backup_folder }}/crontab' 
          remote_src: True 
        when: backup 
```
######note
By default, handlers are executed at the end of the playbook execution, but you can force them to be run when you want using the __meta__ task with the flush_handlers option like: - meta: flush_handlers

### Tasks blocks
```
tasks:
    - block:
       - name: Ensure NTPd is present
       yum:
         name: ntpd
         state: present
       - name: Ensure NTPd is running
       service:
         name: ntpd
         state: started
       enabled: True
     when: ansible_distribution == 'CentOS'
```

#### Defaulting undefined variables
```
{{ backup_disk | default("/dev/sdf") }}
```
other examples
```
{{['a', 'b', 'c', 'd'] | random}}
```
random
```
{{100 | random}}  // 0 to 100
{{50 | random(10)}}  //10 20 30 40 50
{{50 | random(20, 10)}}  //20 30 40 50
```

join strings
```
{{["This", "is", "a", "string"] | join(" ")}}
```

base64
```
{{variable | b64encode}}
{{"aGFoYWhhaGE=" | b64decode}}
```
### Security management
#### Using Ansible vault
we will use the password ansible, so let's start creating a hidden file called .password
```
echo 'ansible' > .password
ansible-vault create secret.yaml
```

other manipulation
```
ansible-vault --vault-password-file=.password edit secret.yaml //first time encryption
ansible-vault --vault-password-file=.password view secret.yaml
ansible-vault --vault-password-file=.password decrypt secret.yaml
ansible-vault --vault-password-file=.password encrypt secret.yaml //re-encrypt
```
rekey, change key file
```
ansible-vault --vault-password-file=.password --new-vault-password-file=.newpassword rekey secret.yaml
```

#### Vaults and playbooks
decrypt
```
ansible-playbook site.yml --vault-password-file .password
```


### Encrypting user passwords
```
vars_prompt:
    - name: ssh_password 
      prompt: Enter ssh_password 
      private: True 
      encryption: md5_crypt 
      confirm: True 
      salt_size: 7 
```
