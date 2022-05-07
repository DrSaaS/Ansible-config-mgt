
** PROJECT: IMPLEMENT WEBSERVERS AND LOAD BALANCER USING ANSIBLE AND JENKINS
---


I created an EC2 Instance (Ubuntu) and named it Jenkins-Ansible. We will use this server to run playbooks.

I opened custom TCP port 8080 on the instance security group

Next, I created a new repository in my GitHub account and named it ansible-config-mgt

The next step was to install Ansible and Jekins on the Jenkins-Ansible server


### Install JDK (since Jenkins is a Java-based application)

``` 
sudo apt update 
```
```
sudo apt install default-jdk-headless
```
### Install Jenkins
```
wget -q -O - https://pkg.jenkins.io/debian-stable/jenkins.io.key | sudo apt-key add -
sudo sh -c 'echo deb https://pkg.jenkins.io/debian-stable binary/ > \
    /etc/apt/sources.list.d/jenkins.list'
```
```
sudo apt update
sudo apt-get install jenkins
```
###Make sure Jenkins is up and running

```
sudo systemctl status jenkins
```

On the Jenkins server, I opened TCP port 8080 by creating a new Inbound Rule in the EC2 Security Group
[Security Group settings](https://missafricagb.com/git/jenkins-ansible-security-group.JPG)

Jenkins url is http://52.56.227.233:8080/

### Install Ansible

```
sudo apt update
sudo apt install ansible
```
Ansible version

```
ansible --version
```

```
ubuntu@ip-172-31-25-140:~$ sudo apt install ansible
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
ansible is already the newest version (2.10.7+merged+base+2.10.8+dfsg-1).
0 upgraded, 0 newly installed, 0 to remove and 26 not upgraded.
ubuntu@ip-172-31-25-140:~$ ansible --version
ansible 2.10.8
  config file = None
  configured module search path = ['/home/ubuntu/.ansible/plugins/modules', '/usr/share/ansible/plugins/modules']
  ansible python module location = /usr/lib/python3/dist-packages/ansible
  executable location = /usr/bin/ansible
  python version = 3.10.4 (main, Apr  2 2022, 09:04:19) [GCC 11.2.0]
```

### The next step is to configure Jenkins build job to save the repository content every time it changes.


I created a new Freestyle project ansible in Jenkins and pointed it to the ‘ansible-config-mgt’ repository.

I Configured Webhook in GitHub and set webhook to trigger the ansible build.

[Github Webhook](https://missafricagb.com/git/github-webhook.JPG)

Next, I configured a Github hook trigger for GITScm polling and a Post-build job in Jenkins to save all (**) files

[Github hook trigger](https://missafricagb.com/git/build-triggers.JPG)

[Post-build job](https://missafricagb.com/git/post-build-job.JPG)

check that artifacts (6th build) are stored on the jenkins server

```
ls /var/lib/jenkins/jobs/ansible/builds/6/archive/

```

[Artifacts on Jenkins Server](https://missafricagb.com/git/files-archived-jenkins-server.JPG)



###The next step was to prepare the Development Environment using VSC

### Next, I connected to the Github repository (Ansible-config-mgt)

Then I proceeded to clone the github repository

[Clone Github repository](https://missafricagb.com/git/vscode-github-repo-clone.JPG)

I confirmed I was in the correct git branch (main) in VSC using gitbash

```
git branch
```

[Git branch confirmation](https://missafricagb.com/git/confirm-git-branch-main.JPG)

projectx
Next I checked the git status


``
git status
```
[Git status](https://missafricagb.com/git/git-status.JPG)

I then created a new branch and named it projectx

```
git checkout -b projectx
```
[New branch](https://missafricagb.com/git/branch-projectx.JPG)


I created a directory and named it playbooks – it will be used to store all the playbook files.
```
mkdir playbooks
``

I also created a directory and named it inventory – it will be used to keep the hosts organised.
```
mkdir inventory
```

Within the playbooks folder, I created the first playbook, and named it common.yml

```
touch playbooks/common.yml
```
Within the inventory folder, I created an inventory file (.yml) for each environment (Development, Staging Testing and Production) dev, staging, uat, and prod respectively.
```
touch inventory/dev.yml inventory/staging.yml inventory/uat.yml inventory/prod.yml
```

### Step 4 - Set up Ansible Inventory
This will define the hosts and group of hosts upon which the playbook commands will operate.

Next I set up the ssh-agent

```
eval `ssh-agent -s`
ssh-add darey-io-keypair.pem
```

I confirmed that the key had been added with the command ssh-add -l
```
ssh-add -l
```

I then connected to the jenkins server using the ssh agent and the servers public IP4 address

```
ssh -A ubuntu@3.9.19.63
```
[SSH-agent connection](https://missafricagb.com/git/ssh-agent-connection.JPG)


I updated inventory/dev.yml with this snippet of code setting each server's private IP as host 

```
[nfs]
172.31.24.237 ansible_ssh_user='ec2-user'

[webservers]
172.31.19.173 ansible_ssh_user='ec2-user'
172.31.27.39 ansible_ssh_user='ec2-user'

[db]
172.31.27.141 ansible_ssh_user='ec2-user' 

[lb]
172.31.29.165 ansible_ssh_user='ubuntu'
```


### CREATE A COMMON PLAYBOOK
---
** Step 5 – Create a Common Playbook
This will give Ansible the instructions on what you needs to be performed on all servers listed in inventory/dev.

I updated playbooks/common.yml file with following code:

```
---
- name: update web, nfs and db servers
  hosts: webservers, nfs, db
  remote_user: ec2-user
  become: yes
  become_user: root
  tasks:
    - name: ensure wireshark is at the latest version
      yum:
        name: wireshark
        state: latest

- name: update LB server
  hosts: lb
  remote_user: ubuntu
  become: yes
  become_user: root
  tasks:
    - name: Update apt repo
      apt: 
        update_cache: yes

    - name: ensure wireshark is at the latest version
      apt:
        name: wireshark
        state: latest

```


**Step 6 – Update GIT with the latest code
The next step was to push the directories and files on the local machine to GitHub.

```
git status

git add .

git commit -m "Commit updated local folders and files"

git push origin projectx
```
















































































































































