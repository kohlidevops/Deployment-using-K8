Step -1: Lauch EC2 Instances with Ubuntu-20

Jenkins Instance with T2.Micro and security group - All Traffic. 

Ansible Instance with T2.Micro and security group - All Traffic.

Kubernetes Instance with T3.Medium and security group - All Traffic

Step -2: Install and configure Jenkins 

SSH to EC2 Instance 

    $sudo su
    #hostnamectl set-hostname jenkins
    #/bin/bash
    #sudo apt-get update -y
    #apt install default-jre -y
    #curl -fsSL https://pkg.jenkins.io/debian-stable/jenkins.io-2023.key | sudo tee \
    /usr/share/keyrings/jenkins-keyring.asc > /dev/null
    echo deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc] \
    https://pkg.jenkins.io/debian-stable binary/ | sudo tee \
    /etc/apt/sources.list.d/jenkins.list > /dev/null
    #sudo apt-get update
    #sudo apt-get install jenkins -y
    #systemctl enable jenkins.service
    #systemctl status jenkins.service

Step -3: To unlock Jenkins

#cat /var/lib/jenkins/secrets/initialAdminPassword

Copy / Paste the password to Jekins console to unlock the Jenkins.

![image](https://github.com/kohlidevops/Deployment-using-K8/assets/100069489/a18ae1d0-1b52-4a5f-920b-c69afbf2c941)

Choose - Install suggested plugins and create a first admin user then save & continue.

![image](https://github.com/kohlidevops/Deployment-using-K8/assets/100069489/3fabc930-128c-4973-918a-6443c023fa4b)

Here we go!

![image](https://github.com/kohlidevops/Deployment-using-K8/assets/100069489/733d8b31-0a8b-42c4-bf28-39940eac58be)

Install a SSH-Agent in Jenkins console - Select - Manage Plugins - Available - SSH-Agent - Install without restart

Step -4: Install and Configure Ansible

SSH to Ansible Instance

    $sudo su
    #hostnamectl set-hostname ansible
    #/bin/bash
    #sudo apt-get update -y
    #sudo apt-add-repository ppa:ansible/ansible -y
    #sudo apt update -y
    #sudo apt install ansible -y
    #ansible --version

Step -5: Install and Configure Kubernetes

SSH to K8 Instance

    $sudo su
    #hostnamectl set-hostname kubernetes
    #/bin/bash
    #sudo apt-get update -y
  
create a docker.sh file and save below command then execute it.

    #nano docker.sh
    #chmod +x docker.sh
    #sh docker.sh

Script to install and configure docker

    #!/bin/bash
    sudo apt update -y
    sudo apt install apt-transport-https ca-certificates curl software-properties-common -y
    curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
    sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu bionic stable" -y
    sudo apt update -y
    apt-cache policy docker-ce -y
    sudo apt install docker-ce -y
    sudo systemctl status docker
    sudo chmod 777 /var/run/docker.sock

Check the docker version and create one more script to install kubernetes

    #docker --version
    #nano minikube.sh
    #sh minikube.sh

    #!/bin/bash
    sudo apt-get update -y
    sudo apt-get install curl wget apt-transport-https virtualbox virtualbox-ext-pack -y
    echo "1st install docker"
    sudo apt update && apt -y install docker.io
    sudo systemctl start docker
    sudo systemctl enable docker
    sudo chmod 666 /var/run/docker.sock
    echo "Apply updates"
    sudo apt update -y 
    sudo apt upgrade -y
    echo " Download Minikube Binary"
    wget https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64
    sudo cp minikube-linux-amd64 /usr/local/bin/minikube
    sudo chmod +x /usr/local/bin/minikube
    minikube version
    echo "Install Kubectl utility"
    curl -LO https://storage.googleapis.com/kubernetes-release/release/`curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt`/bin/linux/amd64/kubectl
    chmod +x kubectl
    sudo mv kubectl /usr/local/bin/
    kubectl version -o yaml
    echo "Start the minikube"
    minikube start 
    minikube status

Step -6: Create a GitHub repository

Login to GitHub and create a remote repository.Then create a Dockerfile using below link.

        https://github.com/kohlidevops/my-k8-project/blob/main/Dockerfile

Step -7: Create a Pipeline project

To Create a new item with Pipeline project.

![image](https://github.com/kohlidevops/Deployment-using-K8/assets/100069489/feece006-20bc-4a76-903b-e2760daa4494)

In your project, select Pipeline syntax to configure the Git checkout stage.

![image](https://github.com/kohlidevops/Deployment-using-K8/assets/100069489/e1ddbe04-ef5c-4197-a46d-ec3d2c9dc830)

Then generate a pipeline syntax

        git branch: 'main', url: 'https://github.com/kohlidevops/my-k8-project.git'

Go back to your project - scroll down - select - Pipeline - Definition - Pipeline script - add Git Checkout stage

        node {
            stage('Git Checkout')
                git branch: 'main', url: 'https://github.com/kohlidevops/my-k8-project.git'
            }

Now Apply & Save, then build your project. The build has been succedded.

![image](https://github.com/kohlidevops/Deployment-using-K8/assets/100069489/c537d3db-ad37-4f72-8147-1f9155849fc5)

Check your Jenkins server through SSH and navigate to inside the below directory

        #cd /var/lib/jenkins/workspace
        #ls

Your project should be available.

Step -8: Configure Git Webhook

We need to configure Git SCM Polling in Jenkins and add webhook in your project in Github to trigger the build stage automatically whenever user changes their code.

Select your project in Jenkins UI and select configure then select the Build triggers - GitHub hook trigger for GITSCM Polling - Apply & Save.

![image](https://github.com/kohlidevops/Deployment-using-K8/assets/100069489/40b19704-f4bc-49e6-a364-aaa0c46b804c)

Then select user in upper-right in jenkins UI - configure - Add API Token - Generate - Copy it to use later

![image](https://github.com/kohlidevops/Deployment-using-K8/assets/100069489/17de3809-9350-432d-885d-24b1d71afa73)

Select your project in GitHub repository - settings - Webhooks - Add Webhook - Copy Paste the API Token here

![image](https://github.com/kohlidevops/Deployment-using-K8/assets/100069489/bad58815-354a-419b-9256-dabdce15dc08)

Events - Let me select individual events - choose - Pushes & Pull requests - Add Webhook

Let's do some changes in Dockerfile and check whether Jenkins build is started automatically.

![image](https://github.com/kohlidevops/Deployment-using-K8/assets/100069489/c85f099f-d15e-4768-8704-a395170461b5)

Perfect! Automatically triggered as we expected.



  

