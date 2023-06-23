Jenkins - Docker - Kubernetes Integration : 

# 1. Install And configure Docker on Ubuntu 18.04 in AWS

sudo apt update -y 
sudo apt install docker.io -y 
sudo systemctl start docker  
sudo systemctl enable docker.service

# 2.Install , start and configure Jenkins 

sudo  apt install openjdk-11-jdk -y
sudo apt install unzip wget -y 

curl -fsSL https://pkg.jenkins.io/debian/jenkins.io-2023.key | sudo tee \
    /usr/share/keyrings/jenkins-keyring.asc > /dev/null

echo deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc] \
    https://pkg.jenkins.io/debian binary/ | sudo tee \
    /etc/apt/sources.list.d/jenkins.list > /dev/n

sudo apt-get update
sudo apt-get install fontconfig openjdk-11-jre
sudo apt-get install jenkins

sudo usermod -aG docker jenkins
sudo systemctl restart docker.service

sudo echo "jenkins ALL=(ALL) NOPASSWD:ALL" | sudo tee /etc/sudoers.d/jenkins

sudo su - jenkins

# 3. Configure Kubernetes Cluster using kops in AWS

 -- Setup Kubernetes (K8s) Cluster on AWS Using KOPS by creating bash scripts using the following command to create a 
 kops.sh file (Vi and paste the following commands)
---------------------------------------------------------------------------------------------------------
#!/bin/bash

#1. install AWS CLI
sudo apt update -y
sudo apt install unzip wget -y 
sudo apt install unzip python3 -y
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
sudo unzip awscliv2.zip
sudo ./aws/install
#aws --version



#2. Install kops software on ubuntu instance:
#Install wget if not installed 
#sudo apt install wget -y
curl -Lo kops https://github.com/kubernetes/kops/releases/download/$(curl -s https://api.github.com/repos/kubernetes/kops/releases/latest | grep tag_name | cut -d '"' -f 4)/kops-linux-amd64
sudo chmod +x kops
sudo mv kops /usr/local/bin/kops

#3. Install kubectl
sudo curl -LO https://storage.googleapis.com/kubernetes-release/release/$(curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt)/bin/linux/amd64/kubectl
sudo chmod +x ./kubectl
sudo mv ./kubectl /usr/local/bin/kubectl
-----------------------------------------------------------------------------------------------------------------------

  -- Run kops.sh as a Jenkins user on the jenkins/kops server

# 4) Create an IAM role from AWS console or CLI with below Policies or attach Admin role to Jenkins/kops EC2 instance.

AmazonEC2FullAccess 
AmazonS3FullAccess 
IAMFullAccess
AmazonVPCFullAccess

or 

# How to  Attach IAM role to Ubuntu server from Console dashbord
1. Select KOPS (Jenkins-Server) Server 
2. Select the action menu on the insatnce dashboard 
3. Click on Security from the menu list 
4. Click on Modify IAM 
5. Select Admin (role) from the dropdown list 
6. Click on Save. 


# 5) Enter command to display a list a resources in your S3 bucket 

aws s3 ls 
 
# 6) Creating S3 bucket and excute below command in KOPS Server, use unique bucket name accross all aws account 

aws s3 mb s3://koftown.local

-- Expose enviromental variables: 
# Add env variable in bashhrc 
vi .bashrc 

# Add the unique name and S3 bucket name that you created to the enviromental variable 
export NAME=koftown.k8s.local
export KOPS_STATE_STORE=s3://koftown.local

# Refresh the bashrc file 
source .bashrc 

# 7) Create sshkey before creating cluster 

ssh-keygen 

# 8) Create kubernetes cluster definfition on S3 bucket 

# first run a imperative command to create cluster 
kops create cluster --zones us-east-1a --networking weave --master-size t2.medium --master-count 1 --node-size t2.medium --node-count=2 ${NAME}

# run this command to view the entire infrastructure
kops get cluster 

# Create a secret key using the following commad 
kops create secret --name ${NAME] sshpublickey admin -i ~/.ssh/id rsa.pub

# 9) Update kubernetes clutser
# Update kops cluster ( After 10 mins cluster will be fully provisioned )
kops update cluster ${NAME} --yes

# 10) Validate your cluster (KOPS will take some time to create cluster and  Execute) 
# To validate cluster run the following command
kops validate cluster ${NAME} --wait 10m 

# 11) To list nodess 

kubectl get nodes

# 12) To Delete Cluster

kops delete cluster --name=${NAME} --state=${KOPS_STATE_STORE} --yes
