
# CONFIGURING ANSIBLE as a BASTON HOST

## INTRODUCTION
In this project, we would be configuring Ansible Client as a Bastion Host also known as Jump Server. A Jump server is an intermediary server through which access to the internal network can be provided. If we should consider the current architecture being worked on, the web servers are always inside a secure network that cannot be reached directly from the internet. This means that even a DevOps engineer cannot ssh into the web servers directly except through a Jump server, this provides better security and reduces attack surface.

## Install and configure Ansible on an EC2 instance.

- Create an EC2 Ubuntu instance
- ssh into the instance
- install Jenkins
- install Ansible
```
sudo apt update
sudo apt install ansible
```
- Check your Ansible version
```
ansible --version
```

- Configure Jenkins build job to save your repository content every time you change it

## Prepare your development environment using Visual Studio Code
- Install Visual Studio Code on your local machine. You can download it from here. As a DevOps engineer the first part is Dev which means you have to be a developer, so you have to be comfortable with your development environment. I would recommend Visual Studio Code as it is free and open source. It is also cross-platform and supports multiple languages.

- After installing Visual Studio Code, configure it to connect to your newly created GitHub repository.

- Then you clone down your ansible-config-mgt repo to your Jenkins-Ansible instance.

```
git clone <ansible-config-mgt repo link>
```
## Begin Ansible Development
- In your ansible-config-mgt repo, create a new branch called ansible-dev and switch to it. This would be used for the development of a new feature.
```
git checkout -b ansible-dev
```

- Create a directory and name it playbooks. This would be used to store all your playbook files.
- Create a directory and name it inventory. This would be used to keep your host organized.
- Within the playbooks folder, create your first playbook and name it common.yml.
- Within the inventory folder, create an inventory file for each environment (Development, Staging, Testing and Production) example dev.yml, staging.yml, uat.yml and prod.yml respectively.


## Set up an Ansible inventory

An Ansible inventory file defines the hosts and groups of hosts upon which commands, modules, and tasks in a playbook operate. Since our intention is to execute Linux commands on remote hosts and ensure that it is the intended configuration on a particular server that occurs. It is important to have a way to organize our hosts in such an Inventory.

To learn how to setup SSH agent and connect VS Code to your Jenkins-Ansible instance, see the video below:

- For Windows users - Set up SSH agent on Windows https://youtu.be/OplGrY74qog

- For Linux users - Set up SSH agent on Linux  https://www.youtube.com/watch?v=RRRQLgAfcJw&t=836s

- Add your private key to the SSH agent

```
ssh-add <path-to-private-key>
```
- Now you confirm that your key has been added to the SSH agent

```
ssh-add -l
```

Now you can connect to your Jenkins-Ansible instance from VS Code using SSH agent. To do this, first install Remote SSH extension, click on the green icon on the bottom left corner of VS Code and select Remote-SSH: Connect to Host... and select your Jenkins-Ansible instance. Before this in the video on setting up SSH-agent, we've set up our ssh config host file to contain the URL to the Jenkins-ansible instance.

Note: For ubuntu instances, the username is ubuntu and for RHEL-based instances, the username is ec2-user.

- Update the inventory/dev.yml file with a snippet of code

```
[nfs]
<NFS-Server-Private-IP-Address> ansible_ssh_user='ubuntu'

[webservers]
<Web-Server1-Private-IP-Address> ansible_ssh_user='ubuntu'
<Web-Server2-Private-IP-Address> ansible_ssh_user='ubuntu'

```

## Create a Common Ansible Playbook

At this stage of the project, it is time to start giving Ansible the instructions on what we need to be performed on all servers listed in inventory/dev.

In the common.yml playbook, we would write configuration for repeatable, reusable, and multi-machine tasks that is common to systems within the infrastructure.

- Now we would update this playbooks/common with the following tasks:
   - Install wireshark on all the servers listed in the inventoty/dev.yml file
   - Create a directory and a file inside it
   - Change the timezone on all servers
   - Run some shell script


- name: update web, nfs
  hosts: webservers, nfs, 
  remote_user: ubuntu
  become: yes
  become_user: root
  tasks:
    - name: ensure wireshark is at the latest version
      yum:
        name: wireshark
        state: latest

    - name: create a directory
      file:
        path: /home/ubuntu/test
        state: directory

    - name: create a file
      file:
        path: /home/ubuntu/test/test.txt
        state: touch

    - name: change timezone
      command: timedatectl set-timezone Africa/Lagos

    - name: run a shell script
      shell: |
        echo "Hello World" > /home/ubuntu/test/test.txt


## Update git with your changes
At the moment all our directories and files are local to our machine. We need to push them to our remote repository on GitHub.

```
git status
git add .
git commit -m "update common playbook"
git push origin <branch-name>
```

- Once your code changes appear in the main branch Jenkins will do its job and save all the files (build artifacts) to

```
/var/lib/jenkins/jobs/ansible/builds/<build_number>/archive/
```
## Run First Ansible Test

Now it is time to execute ansible-playbook command and verify that the playbook is working as expected.

- SSH into your Jenkins-Ansible server and run the following command

```
cd ansible-config-mgt
ansible-playbook -i inventory/dev.yml playbooks/common.yml
```

- To confirm whether all the installations were successful, SSH into your servers and run the following commands

```
wireshark --version
```
Now we've successfully been able to configure ansible as a bastion host and run a playbook on all servers in our infrastructure.


