# Create_Ansible_Playbooks_To_Manage_Nginx_Webservers
# Steps
## Create EC2 instances (using UBUNTU here)
- Create One Ansible-master with a newly created key Ansible-master-key.pem 
- Create three Slave servers named server-1, server-2, server-3
## Connect Ansible-master
- Open an SSH client.
- Go to your private key file. here is downloads file on the local machine. 
- Run this command, if necessary, to ensure your key is not publicly viewable.
 chmod 400 Ansible-master-key.pem
- Connect to your instance using its Public DNS:
 ec2-3-84-91-184.compute-1.amazonaws.com
- Example:
 ssh -i "Ansible-master-key.pem" ubuntu@ec2-3-84-91-184.compute-1.amazonaws.com

## Execute commands
 - sudo apt-add-repository ppa:ansible/ansible 
 - sudo apt update
 - sudo apt install ansible 
 - ansible --version
 - sudo vim /etc/ansible/hosts
## make group in vim
 [servers]
 server-1 ansible_host=12.220.44.91
 server-2 ansible_host=18.220.101.84
 server-3 ansible_host=101.25.40.101
Esc
:wq!
 
## create key directory
- mkdir keys
- cd keys

## copy source using scp
- go to local downloads
scp -i "Ansible-master-key.pem" Ansible-master-key.pem ubuntu@ec2-3-90-57-129.compute-1.amazonaws.com:/home/ubuntu/keys
- open ssh client 
- pwd shoud in keys directory, make assured it
  ls (see Ansible-master-key.pem is copied and present)

## create variables in vim, in keys directory
- vim /etc/ansible/hosts
 - [servers]
 - server-1 ansible_host=12.220.44.91
 - server-2 ansible_host=18.220.101.84
 - server-3 ansible_host=101.25.40.101
 - [servers:vars]
 - ansible_python_interpreter=/usr/bin/python3
 - ansible_user=ubuntu
 - ansible_ssh_private_key_file=/home/ubuntu/keys/Ansible-master-key.pem
- Esc
- :wq!

## ping ansible servers
 - ansible servers -m ping
 - yes
 - sever-1 connected
 - yes
 - sever-2 connected
 - yes
 - server-3 connected

## check then update all the servers with ad hoc commands
 - ansible servers -a "free -h"
 - ansible servers -a "sudo apt update"

 ## only one server update
 - stay in keys directory 
 - sudo vim /etc/ansible/hosts

 - [servers]
 - server-1 ansible_host=12.220.44.91
 - server-2 ansible_host=18.220.101.84
 
 - [prd]
 - server-3 ansible_host=101.25.40.101

 - [all:vars]
 - ansible_python_interpreter=/usr/bin/python3
 - ansible_user=ubuntu
 - ansible_ssh_private_key_file=/home/ubuntu/keys/Ansible-master-key.pem
 - Esc
 - :wq!
 - ansible-inventory --list -y
 - ansible prd -m ping

## see success printed as below
 server-3 | SUCCESS => {
 "changed": false,
 "ping": "pong"
 }

# Create Playbooks
 - cd .. 
 - mkdir playbooks
 - cd playbooks
 - vim date_play.yml
  -
   - name: Date Playbook
   - hosts: servers
   - tasks: 
     - name: Show uptime
     - command: uptime

- Esc
- :wq!

ansible-playbook -v date_play.yml

# Nginx installation
 stay at playbooks directory
 vim install_nginx_play.yml
  - 
    name: Install Nginx and start it 
    hosts: servers
    become:  yes
    tasks: 
      - name: Install Nginx
        apt: 
          name: nginx 
          state: latest

 Esc 
:wq!

ansible-playbook install_nginx_play.yml

## Start Nginx
 stay at playbooks directory
 vim install_nginx_play.yml
  - 
    name: Install Nginx and start it 
    hosts: servers
    become:  yes
    tasks: 
      - name: Install Nginx
        apt: 
          name: nginx 
          state: latest
      - name: Start Nginx
        service:
          name: nginx 
          state: started
          enabled: yes 

 Esc 
:wq!

ansible-playbook install_nginx_play.yml

## Go to aws ec2 console
 - copy ip address from any sever, maybe 1, 2, 3
 - open a new window 
 - paste that ip address 

 # SUCCESS !!
 ## Nginx Server is successfully installed and working, NOW !! Thank YOU
