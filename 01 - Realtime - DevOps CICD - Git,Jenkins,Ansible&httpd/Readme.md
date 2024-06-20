# DevOps Project: CI/CD Pipeline with Git, Jenkins, Ansible, and HTTPD Server

## Table of Contents
1. [Introduction](#introduction)
2. [Technologies Used](#technologies-used)
3. [Project Architecture](#project-architecture)
4. [Setup and Installation](#setup-and-installation)
5. [Step-by-Step Process](#step-by-step-process)
6. [Screenshots](#screenshots)
7. [Conclusion](#conclusion)
8. [Contact Information](#contact-information)

## Introduction
This project demonstrates a Continuous Integration and Continuous Deployment (CI/CD) pipeline using Git, Jenkins, Ansible, and an HTTPD server. The pipeline automates the deployment of a web application, ensuring consistent and reliable delivery.

## Technologies Used
- **Git**: Version control
- **Jenkins**: Continuous integration server
- **Ansible**: Configuration management and orchestration tool
- **HTTPD (Apache)**: Web server

## Project Architecture
![](Images/project_architecture.jpg)

## Setup and Installation

### Step 1: Launch Instances
- Launch instances with Amazon Linux 2.
- Connect using Putty/MobaXterm.
- Modify instance security inbound rules to allow custom TCP 8080.

![EC2 Instances](Images/EC2_Instances.jpg)
![Servers](Images/servers.jpg)
![Inbound Rules](Images/Inbound_Rule.jpg)

### Step 2: Configure Git and Jenkins

**Git Installation**
1. Switch to root user: `sudo -i` or `sudo su`
2. Update packages: `sudo yum update`
3. Install Git: `sudo yum install git`
4. Verify Git installation: `git --version`

**Jenkins Installation**
1. Switch to root user: `sudo su -`
2. Add Jenkins repo: `sudo wget -O /etc/yum.repos.d/jenkins.repo https://pkg.jenkins.io/redhat-stable/jenkins.repo`
3. Import Jenkins key: `sudo rpm --import https://pkg.jenkins.io/redhat-stable/jenkins.io-2023.key`
4. Install Java 11: `amazon-linux-extras install java-openjdk11 -y`
5. Install Jenkins: `yum install jenkins -y`
6. Enable and start Jenkins: 
    ```sh
    systemctl enable jenkins
    systemctl start jenkins
    systemctl status jenkins
    ```
7. Retrieve Jenkins initial admin password: `cat /var/lib/jenkins/secrets/initialAdminPassword`
8. Access Jenkins UI: `http://<Public-IPv4-address>:8080/` and complete the setup.
9. Complete the basic user & default configurations and you will be launced to Jenkins UI.

### Step 3: Install Ansible
1. Launch an EC2 instance and connect using Putty/mobaXterm.
2. Switch to root user.
3. Update packages: `sudo yum update -y`
4. Install Ansible: `sudo amazon-linux-extras install ansible2 -y`
5. Verify Ansible installation: `ansible --version`

### Step 4: Configure Jenkins and GitHub Integration
1. In Jenkins, generate API token: Jenkins -> User -> Configure -> API token -> Generate and save it.
2. In GitHub, create a new repository and add a webhook: Settings -> Webhooks -> Add webhook -> Paste Jenkins API token into the secret field.
3. Install Jenkins plugins: Manage Jenkins -> Plugins -> Available plugins -> Install "Publish Over SSH" plugin and restart Jenkins.

![API Token](Images/apitoken_jenkins.jpg)
![Webhook-Git](Images/Webhook_Git.jpg)
![Publish Over SSH Plugin](Images/publish_over_ssh_plugin.jpg)

### Step 5: Establish SSH Connections Between Machines
1. Following steps should be done in all 3 machines
2. Publish the connection between 3 vm's : Jenkins-ansible-webserver by using ssh
3. Set root password: `passwd root`
4. Edit SSH configuration: `vi /etc/ssh/sshd_config`
    - Uncomment `PermitRootLogin yes`
    - Change `PasswordAuthentication no` to `yes`
5. Restart and enable SSH: 
    ```sh
    systemctl restart sshd
    systemctl enable sshd
    ```
6. Test SSH connections: `ssh root@<private IP>`
7. - Click enter and enter password, it will connect to other machine
   - Once connection is established, use command exit to come out from session

### Step 6: Generate and Copy SSH Keys
1. Generate SSH keys: `ssh-keygen`
2. Copy SSH key to Ansible server: `ssh-copy-id -i root@<private IP of Ansible server>`
3. Verify connection and exit: `ssh root@<private IP of Ansible server>`

### Step 7: Create Ansible Playbook
1. Create playbook: `vi sample.yaml`
    ```yaml
    - name: Install and configure Apache
      hosts: all
      become: true

      tasks:
        - name: Install httpd
          yum:
            name: httpd
            state: present

        - name: Start httpd service
          service:
            name: httpd
            state: started

        - name: Copy index.html to web server root
          copy:
            src: /opt/index.html
            dest: /var/www/html/index.html
    ```
2. Edit Ansible hosts file: `vi /etc/ansible/hosts`
    ```ini
    [web]
    <ipaddress>
    ```
3. Ping web server: `ansible -m ping <private IP of web server>`
4. Run playbook: `ansible-playbook sample.yaml -c ssh`

### Step 8: Push Code to GitHub
1. Create directory and initialize Git: 
    ```sh
    mkdir cicd
    cd cicd
    git init
    ```
2. Create and add file: 
    ```sh
    vi index.html
    git add index.html
    git commit -m "first commit"
    git branch -m master
    git remote add origin https://github.com/Charan-DevOps-Git/cicd.git
    git push origin master
    ```
3. File will be pushed to master branch, verify it on GitHub

![GitHub Repo](Images/github_repo.jpg)
![Access Token](Images/Personal_Access_Token.jpg)

### Step 9: Configure Jenkins & Ansible SSH Server
1. Add SSH server: Jenkins -> Dashboard -> System -> SSH Server.
2. Test connection with Jenkins and Ansible servers.

![Jenkins](Images/SSH_Server_Jenkins.jpg)
![Ansible](Images/SSH_Server_Ansible.jpg)

### Step 10: Create Jenkins Pipeline
1. Source Code Management: Git.
2. Build Triggers: GitHub hook trigger for GITScm polling.
3. Build Steps: Send files or execute commands over SSH.
    - Jenkins Exec command: `rsync -avh /var/lib/jenkins/workspace/cicdtest/index.html root@<private IP>:/opt/`
4. Save and build.
5. Verify deployment in Ansible VM: `ls -l /opt/`

![SCM](Images/SCM_Git.jpg)
![Build Step](Images/build_step_jenkins.jpg)
![Jenkins Pipeline](Images/Project_Jenkins.jpg)

### Step 11: Add Post Build Actions
1. Add post build actions: Execute Ansible playbook `sample.yaml`.
2. Save and build.
3. Access web server public IP to see the deployed HTML file.

![Post Build Action](Images/PostBuild_Ansible.jpg)

### Step 12: Test CI/CD Pipeline
1. Modify code in GitHub to trigger the build.
2. Verify updates in web server IP address.

![Web Server Output](Images/WebServer_Output.jpg)

## Console Output

![Build-1](Images/output-1.jpg)
![Build-2](Images/output-2.jpg)
![Build-3](Images/output-3.jpg)

## Conclusion
This project demonstrates the successful implementation of a CI/CD pipeline using Git, Jenkins, Ansible, and an HTTPD server. It automates the deployment process, ensuring efficient and reliable application delivery.

## Contact Information
For any queries, feel free to contact me at [sscharan95@gmail.com].