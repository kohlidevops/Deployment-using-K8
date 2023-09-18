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

Step -4: Install and Configure Ansible & Docker

SSH to Ansible Instance

    $sudo su
    #hostnamectl set-hostname ansible
    #/bin/bash
    #sudo apt-get update -y
    #sudo apt-add-repository ppa:ansible/ansible -y
    #sudo apt update -y
    #sudo apt install ansible -y
    #ansible --version
    #sudo apt-get install docker.io -y
    #sudo systemctl start docker.service
    #sudo systemctl enable docker.service
    #sudo systemctl status docker.service

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

Step -9: Configure Passwordless authentication in Ansible server

SSH to Ansible machine and edit sshd config file and update the line like below

        #vi /etc/ssh/sshd_config

        PermitRootLogin yes
        PasswordAuthentication yes

        save and exit and restart the SSHD

        #service sshd restart

To set the password for ubuntu user in Ansible machine

        #sudo passwd ubuntu
        
SSH to Jenkins machine and generate ssh-key and copy it to the Ansible machine

        #ssh-keygen

![image](https://github.com/kohlidevops/Deployment-using-K8/assets/100069489/0b7a80b6-eb32-430b-944e-ba17fb8d3005)

        #ssh-copy-id ubuntu@ansible-machine-private-ip
        #ssh ubuntu@ansible-machine-private-ip

Perfect! I can able to login to ansible machine from jenkins machine. Expected one.

Step -10: Configure ansible stage in Jenkins UI

Go back to Pipeline syntax generator - select - sshagent-SSH Agent

![image](https://github.com/kohlidevops/Deployment-using-K8/assets/100069489/f9d3b7b1-ade9-47e6-a9a1-99a89ee4d7a8)

![image](https://github.com/kohlidevops/Deployment-using-K8/assets/100069489/22a1c7cf-1ace-471d-baf9-6319750353bd)

Enter the private key directly - then Add this configuration to generate the syntax

        sshagent(['ansible_ssh']) {
            // some block
            }

Now you Pipeline stage would be like below

        node {
            stage('Git Checkout'){
                git branch: 'main', url: 'https://github.com/kohlidevops/my-k8-project.git'
                }
            stage('Sending Docker file to Ansible Server Over SSH'){
                sshagent(['ansible_ssh']) {
                sh 'ssh -o StrictHostKeyChecking=no ubuntu@172.31.3.201'
                sh 'scp /var/lib/jenkins/workspace/my-k8-project/* ubuntu@172.31.3.201:/home/ubuntu/'
                        }
                    }    
            }

Apply & save then start the build. Go back to ansible machine and check whether Dockerfile is available inside /home/ubuntu/

Yes, File is available.

![image](https://github.com/kohlidevops/Deployment-using-K8/assets/100069489/6ae7effc-f0f9-4b90-bb09-49f9fdb21d36)

Step -11: Build and Tagging stage

Go back to your project in Jenkins UI and choose - configure - Pipeline statement. Now your Pipeline would be like below.

        node {
            stage('Git Checkout'){
                git branch: 'main', url: 'https://github.com/kohlidevops/my-k8-project.git'
                    }
            stage('Sending Docker file to Ansible Server Over SSH'){
                sshagent(['ansible_ssh']) {
                    sh 'ssh -o StrictHostKeyChecking=no ubuntu@172.31.3.201'
                    sh 'scp /var/lib/jenkins/workspace/my-k8-project/* ubuntu@172.31.3.201:/home/ubuntu/'
                            }
                    }
             stage('Docker Build Image'){
                sshagent(['ansible_ssh']){
                    sh 'ssh -o StrictHostKeyChecking=no ubuntu@172.31.3.201 cd /home/ubuntu'
                    sh 'ssh -o StrictHostKeyChecking=no ubuntu@172.31.3.201 sudo docker image build -t $JOB_NAME:v1.$BUILD_ID .'
                                }
                    }
            stage('Docker image tagging'){
                sshagent(['ansible_ssh']){
                     sh 'ssh -o StrictHostKeyChecking=no  ubuntu@172.31.3.201 cd /home/ubuntu'
                     sh 'ssh -o StrictHostKeyChecking=no  ubuntu@172.31.3.201 sudo docker image tag $JOB_NAME:v1.$BUILD_ID latchudevops/$JOB_NAME:v1.$BUILD_ID'
                     sh 'ssh -o StrictHostKeyChecking=no  ubuntu@172.31.3.201 sudo docker image tag $JOB_NAME:v1.$BUILD_ID latchudevops/$JOB_NAME:latest'
                        }
                }
            }

Apply & Save - Start the build. Yes build has been succedded.

![image](https://github.com/kohlidevops/Deployment-using-K8/assets/100069489/919e44ad-da3b-4ca6-b604-dc94f3a97dd5)

Let's go and check with ansible machine whether images are created.

        #sudo docker images

![image](https://github.com/kohlidevops/Deployment-using-K8/assets/100069489/05da7216-a839-4460-bf59-75488128c80b)

Step -12: Pushing Docker image to Docker Hub

Go back to Jenkins pipeline syntax generator to create a syntax to login docker hub. Select - withCredentials: Bind credentials to varibales with Secret text - then Add Jenkins to inject docker hub password.

![image](https://github.com/kohlidevops/Deployment-using-K8/assets/100069489/ca3b0e0e-4858-43f2-bbc4-b3eed18fdeff)

![image](https://github.com/kohlidevops/Deployment-using-K8/assets/100069489/8bc4824d-53c0-4329-aaf2-e45944da49ed)

Then generate the syntax.

        withCredentials([string(credentialsId: 'docker_hub_password', variable: 'docker_hub_password')]) {
            // some block
            }

Go to your Jenkins project - configure - pipeline statement. Your Pipeline script would be like below

        node {
            stage('Git Checkout'){
                git branch: 'main', url: 'https://github.com/kohlidevops/my-k8-project.git'
                        }
            stage('Sending Docker file to Ansible Server Over SSH'){
                sshagent(['ansible_ssh']) {
                    sh 'ssh -o StrictHostKeyChecking=no ubuntu@172.31.3.201'
                    sh 'scp /var/lib/jenkins/workspace/my-k8-project/* ubuntu@172.31.3.201:/home/ubuntu/'
                                }
                        }
             stage('Docker Build Image'){
                sshagent(['ansible_ssh']){
                    sh 'ssh -o StrictHostKeyChecking=no ubuntu@172.31.3.201 cd /home/ubuntu'
                    sh 'ssh -o StrictHostKeyChecking=no ubuntu@172.31.3.201 sudo docker image build -t $JOB_NAME:v1.$BUILD_ID .'
                                }
                        }
            stage('Docker image tagging'){
                sshagent(['ansible_ssh']){
                     sh 'ssh -o StrictHostKeyChecking=no  ubuntu@172.31.3.201 cd /home/ubuntu'
                     sh 'ssh -o StrictHostKeyChecking=no  ubuntu@172.31.3.201 sudo docker image tag $JOB_NAME:v1.$BUILD_ID latchudevops/$JOB_NAME:v1.$BUILD_ID'
                     sh 'ssh -o StrictHostKeyChecking=no  ubuntu@172.31.3.201 sudo docker image tag $JOB_NAME:v1.$BUILD_ID latchudevops/$JOB_NAME:latest'
                                }
                        }
            stage ('Push Docker Images to DockerHub'){
                 sshagent(['ansible_ssh']) {
                     withCredentials([string(credentialsId: 'docker_hub_password', variable: 'docker_hub_password')]) {
                         sh 'ssh -o StrictHostKeyChecking=no  ubuntu@172.31.3.201 sudo docker login -u latchudevops -p $docker_hub_password'
                         sh 'ssh -o StrictHostKeyChecking=no  ubuntu@172.31.3.201 sudo docker image push latchudevops/$JOB_NAME:v1.$BUILD_ID'
                         sh 'ssh -o StrictHostKeyChecking=no  ubuntu@172.31.3.201 sudo docker image push latchudevops/$JOB_NAME:latest'
                                }
                        }
                    }

        }


Apply & save - Then start the build. Yes! the build has been succedded.

![image](https://github.com/kohlidevops/Deployment-using-K8/assets/100069489/861d6e3d-57a0-4384-ac93-24fef548c3d1)

Yes! I can able to see the latest & version of my images in docker hub repository.

![image](https://github.com/kohlidevops/Deployment-using-K8/assets/100069489/677104c0-fa8a-4e9f-b8e0-b38904a3150d)

Step -13: Create a Deployment, Service and Ansible yaml files

Create a Deployment yaml file - copy paste the file using below link.
