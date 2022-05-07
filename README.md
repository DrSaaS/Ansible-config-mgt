
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










































