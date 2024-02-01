## What is Ansible?

<p> Ansible is an open-source software provisioning configuration management, and application-deployment tool. It runs on many Unix-like systems, and can configure both Unix-like systems as well as Microsoft Windows. It includes its own declarative language to describe system configuration. </p>


## Why Ansible?

<li> Brings hugs time saveing </li>

<li> Agentless</li>

<li> Playbooks are easy to read and edit </li>

<li> It is written in Python </li>

<li> It is open source </li>


## Here are the steps for install ansbile in ubuntu 22.04.

```
## commands without prompting for the password then run the following echo and tee command on each managed host

echo "sysops ALL=(ALL) NOPASSWD:ALL" | sudo tee /etc/sudoers.d/sysops

```

### 1) Apply Updates.

```
sudo apt update
sudo apt upgrade -y
```


### 2) Add Ansible PPA Repository.

```
sudo apt install -y software-properties-common
sudo add-apt-repository --yes --update ppa:ansible/ansible

sudo apt update
```

### 3) Install Ansible.

```
sudo apt install -y ansible

ansible --version

# If showing error this one..
ERROR: Ansible requires the locale encoding to be UTF-8; Detected ISO8859-1.


export LANG=en_US.UTF-8


ansible --version
```

### 4) Setup SSH keys and Share it Among Managed Nodes.

```
ssh-keygen

```

### 5). Now, Copy the ssh public key into the server.

```
ssh-copy-id -i ~/.ssh/id_rsa.pub redhat@192.168.122.212
```

### 6). Now, Add the server host details in the ansible hosts.

```
vim /etc/ansible/hosts

192.168.122.234 ansible_connection=ssh ansible_ssh_user=redhat ansible_ssh_pass='123' ansible_sudo_pass='123' ansible_become=true


```

### 7). Let’s verify the Server connectivity using ping commands.

```
ansible all -m ping

## Success messages..
192.168.122.212 | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python3"
    },
    "changed": false,
    "ping": "pong"
}
```

### 8). Test Ansible Installation.

```
cat rhel-server.yaml 
---
- name: Playbook to Install Packages
  hosts: 192.168.122.212
  become: yes  # This allows tasks to run as a privileged user (sudo)

  tasks:
    - name: Install php and mariadb
      package:
        name:
          - php
          - mariadb-server
        state: present
      
```

### 9. Now, Run the ansible and install the package on server.

```
ansible-playbook rhel-server.yaml
```

----------------------------------------------

### 10. Now, Create Ansible.cfg File and Setup Inventory


```
mkdir Inventory
cd Inventory/

wget https://raw.githubusercontent.com/ansible/ansible/stable-2.11/examples/ansible.cfg

vim ansible.cfg

## Under the default sections.
[defaults]
inventory      = /home/user/Inventory/server
remote_user =  redhat
host_key_checking = False

## Under the privilege_escalation section.
become=True
become_method=sudo
become_user=root
become_ask_pass=False

```
### 11. Now, Enter the details for servers.

```
vim server

cat server 
# [dev]
# node2.example.com

[prod]
192.168.122.212

## Note: change ther server user permission.
sudo visudo

## Allow root to run any commands anywhere 
root    ALL=(ALL)       ALL
redhat ALL=(ALL) NOPASSWD: ALL


```

## Now, Verify the server connection.

```
ansible all -m ping

192.168.122.212 | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python3"
    },
    "changed": false,
    "ping": "pong"
}

```

### Now, Create ansible yaml file using inventory server details.

cat packages.yaml 
---
- name: Playbook to Install Packages
  hosts:
    - prod
  tasks:
  - name: Install php and mariadb
    package:
      name:
        - php
        - mariadb-server
      state: present


### Now, Run this playbook..

```
ansible-playbook packages.yaml

```

### Install vs code on rhel9.1 server using playbook.

```
 cat new-vs.yaml 
---
- name: Install VS Code on RHEL 9.1
  hosts: 
    - prod
  become: yes

  tasks:
    - name: Install required dependencies
      dnf:
        name:
          - libX11
        state: present
      become: yes

    - name: Download VS Code RPM package
      get_url:
        url: "https://go.microsoft.com/fwlink/?LinkID=760868"
        dest: "/tmp/code.rpm"
        validate_certs: no

    - name: Import GPG key for VS Code
      rpm_key:
        key: "https://packages.microsoft.com/keys/microsoft.asc"
        state: present
      become: yes

    - name: Install VS Code from RPM
      dnf:
        name: "/tmp/code.rpm"
        state: present
        disable_gpg_check: yes
      become: yes

    - name: Clean up downloaded RPM
      file:
        path: "/tmp/code.rpm"
        state: absent



```

 




