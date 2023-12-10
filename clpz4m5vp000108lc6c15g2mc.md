---
title: "Secure Swiggy Clone App Deployment on AWS: Building a DevSecOps Pipeline with Terraform, Jenkins, SonarQube, Trivy, Argocd, and EKS."
datePublished: Sun Dec 10 2023 06:51:13 GMT+0000 (Coordinated Universal Time)
cuid: clpz4m5vp000108lc6c15g2mc
slug: secure-swiggy-clone-app-deployment-on-aws-building-a-devsecops-pipeline-with-terraform-jenkins-sonarqube-trivy-argocd-and-eks
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1702176484541/08bb7986-cefc-498d-8153-2e073f9a4cf0.jpeg
ogImage: https://cdn.hashnode.com/res/hashnode/image/upload/v1702190510372/36d01251-8822-4547-ac36-7349c4cb0de3.jpeg
tags: devsecops-pipeline-swiggy-clone-app

---

### Introduction:

In today's fast-paced digital landscape, building and deploying applications not only requires speed but also airtight security. That's where DevSecOps comes into play, blending development, security, and operations into a single, unified process.

In this guide, we will embark on a journey where we'll leverage Terraform, Jenkins CI/CD, SonarQube, Trivy, Argocd, and Amazon Elastic Kubernetes Service (EKS) to create a robust and secure pipeline for deploying applications on Amazon Web Services (AWS).

Whether you're an experienced developer looking to enhance your DevSecOps skills or a newcomer eager to explore this exciting intersection of software development and security, this guide has something valuable to offer.

Let's dive in and explore the steps to safeguard your Amazon app while ensuring smooth and efficient deployment.

### Architecture:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1702180339650/c649f1d2-ba58-4fd2-a833-e2e58e3eccb9.jpeg align="center")

### Overview:

* **Infrastructure as Code:** Use Terraform to define and manage AWS infrastructure for the application.
    
* **Container Orchestration:** Utilize Amazon EKS for managing and scaling containerized applications.
    
* **CI/CD with Jenkins:** Set up Jenkins to automate building, testing, and deploying the application.
    
* **Static Code Analysis:** Incorporate SonarQube to analyze code quality and identify vulnerabilities.
    
* **Container Image Scanning:** Integrate Trivy to scan container images for security issues.
    
* **Application Deployment with Argocd:** Use Argocd for declarative, GitOps-based application deployment on EKS.
    

### Project Resources:

**GitHub Link:**

[https://github.com/sridhar-modalavalasa/Swiggy-Clone-App](https://github.com/sridhar-modalavalasa/Swiggy-Clone-App) - Application code.

[https://github.com/sridhar-modalavalasa/Swiggy-App-ArgoCD](https://github.com/sridhar-modalavalasa/Swiggy-App-ArgoCD) - Manifest files.

### Step1: Create IAM User

Navigate to the **AWS console**

* Create an **IAM** user with **administration access.**
    
* Log in to the AWS Console with the above user.
    
* Create one free-tier EC2 instance with Ubuntu.
    
* Login to the EC2 instance and follow the below steps.
    

### **Step2: Aws Configuration**

Install the AWS Cli in your EC2 Ubuntu.

**Install AWS CLI:**

```plaintext
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
unzip awscliv2.zip
sudo ./aws/install
aws --version
```

```plaintext
aws configure
```

Provide your Aws **Access key** and **Secret Access key**

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1702182829762/e1f93ce4-8571-4075-a1fd-810f66f0d33e.png align="left")

### **Step3: Terraform files and Provision Jenkins, sonarqube**

**Terraform Installation in an EC2 Instance:**

```plaintext
wget https://releases.hashicorp.com/terraform/1.3.7/terraform_1.3.7_linux_amd64.zip
unzip terraform_1.3.7_linux_amd64.zip
mv terraform /usr/local/bin
sudo mv terraform /usr/local/bin
terraform -v
```

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1702183229643/8d789cf7-9481-40bf-ad68-f477f17ea075.png align="center")

**main.tf**

```plaintext
resource "aws_instance" "web" {
  ami                    = "ami-0fc5d935ebf8bc3bc"   #change ami id for different region
  instance_type          = "t2.large"
  key_name               = "key"
  vpc_security_group_ids = [aws_security_group.Jenkins-sg.id]
  user_data              = templatefile("./install.sh", {})

  tags = {
    Name = "Jenkins-sonarqube-trivy-vm"
  }

  root_block_device {
    volume_size = 30
  }
}

resource "aws_security_group" "Jenkins-sg" {
  name        = "Jenkins-sg"
  description = "Allow TLS inbound traffic"

  ingress = [
    for port in [22, 80, 443, 8080, 9000, 3000] : {
      description      = "inbound rules"
      from_port        = port
      to_port          = port
      protocol         = "tcp"
      cidr_blocks      = ["0.0.0.0/0"]
      ipv6_cidr_blocks = []
      prefix_list_ids  = []
      security_groups  = []
      self             = false
    }
  ]

  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }

  tags = {
    Name = "jenkins-sg"
  }
}
```

**provider.tf**

```plaintext
#provider.tf

terraform {
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.0"
    }
  }
}

# Configure the AWS Provider
provider "aws" {
  region = "us-east-1"  #change your region
}
```

**install.sh**

This will install Jenkins, Docker, Sonarqube, and Trivy by Terraform with an EC2 instance.

```plaintext
#!/bin/bash
sudo apt update -y
wget -O - https://packages.adoptium.net/artifactory/api/gpg/key/public | tee /etc/apt/keyrings/adoptium.asc
echo "deb [signed-by=/etc/apt/keyrings/adoptium.asc] https://packages.adoptium.net/artifactory/deb $(awk -F= '/^VERSION_CODENAME/{print$2}' /etc/os-release) main" | tee /etc/apt/sources.list.d/adoptium.list
sudo apt update -y
sudo apt install temurin-17-jdk -y
/usr/bin/java --version
curl -fsSL https://pkg.jenkins.io/debian-stable/jenkins.io-2023.key | sudo tee /usr/share/keyrings/jenkins-keyring.asc > /dev/null
echo deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc] https://pkg.jenkins.io/debian-stable binary/ | sudo tee /etc/apt/sources.list.d/jenkins.list > /dev/null
sudo apt-get update -y
sudo apt-get install jenkins -y
sudo systemctl start jenkins
sudo systemctl status jenkins

#install docker
sudo apt-get update
sudo apt-get install docker.io -y
sudo usermod -aG docker ubuntu  
newgrp docker
sudo chmod 777 /var/run/docker.sock
docker run -d --name sonar -p 9000:9000 sonarqube:lts-community

#install trivy
sudo apt-get install wget apt-transport-https gnupg lsb-release -y
wget -qO - https://aquasecurity.github.io/trivy-repo/deb/public.key | gpg --dearmor | sudo tee /usr/share/keyrings/trivy.gpg > /dev/null
echo "deb [signed-by=/usr/share/keyrings/trivy.gpg] https://aquasecurity.github.io/trivy-repo/deb $(lsb_release -sc) main" | sudo tee -a /etc/apt/sources.list.d/trivy.list
sudo apt-get update
sudo apt-get install trivy -y
```

**Terraform commands to provision:**

```plaintext
terraform init
```

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1702183275652/367086c4-fd6a-43a5-ac7c-3c2ca0b41340.png align="center")

```plaintext
terraform validate
```

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1702183322819/531a79b5-d73a-4701-a40f-e994f6615ed7.png align="center")

```plaintext
terraform plan
```

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1702183358540/2fd34c63-8a06-483e-a775-e37782268b91.png align="center")

```plaintext
terraform apply
```

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1702183391953/24d8d871-c11e-42a9-8296-43dac42ba488.png align="center")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1702183408740/d252cab4-a511-404b-b616-4bcb2ea95e40.png align="center")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1702183505781/94b500ef-ec65-4032-bef8-84bf625f4db9.png align="center")

The EC2 instance Jenkins-sonarqube-trivy-vm is created by Terraform with Jenkins, Sonarqube, and Trivy as userdata for the EC2 instance, which is installed during the creation of the EC2 instance.

**Output:**

Take the public IP address of the EC2 instance, as shown in the below image.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1702183862783/05d315d7-dccd-4928-8285-c7806aca228c.png align="center")

```plaintext
<Public IPV4 address>:8080. #For accessing Jenkins
```

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1702183941268/3b474844-fc37-4354-ad4d-abc6197dfeff.png align="center")

```plaintext
sudo cat /var/lib/jenkins/secrets/initialAdminPassword
```

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1702183979524/42dd14bd-b004-4825-b7a8-6d704e0927a2.png align="center")

Unlock Jenkins using an administrative password and install the suggested plugins.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1702184009718/220fd0e7-8677-45b0-bcd4-864ca6140f0f.png align="center")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1702184057274/24c6d381-fd6a-480f-9765-6c193b18f9b3.png align="center")

Jenkins will now get installed and install all the libraries.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1702184089284/2b9472b9-dd89-43e5-8722-d340d4fc2263.png align="center")

Create a user, click save, and continue.

Jenkins Getting Started Screen.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1702184154349/77e112a9-9681-4a30-9bb4-4da534f71347.png align="center")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1702184130279/5d0a2280-9315-44e0-b2c7-667924e1893c.png align="center")

Copy your public key again and paste it into a new tab.

```plaintext
<instance-public-ip>:9000
```

Now our Sonarqube is up and running as a Docker container.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1702185592435/1057a239-def8-4f10-a76c-4f768ff9d348.png align="center")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1702184238651/631c34b0-b4de-4ae1-a9e4-6c8d36baf169.png align="center")

Enter your username and password, click on login, and change your password.

```plaintext
username admin
password admin
```

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1702184287966/3fa0b289-9c08-4e2c-a07e-61db25ccd699.png align="center")

Update the new password. This is the Sonar Dashboard, as shown below.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1702184334262/85d152eb-a13c-421c-a469-52ea229f9648.png align="center")

**Check Trivy version**

Check the Trivy version in an Ec2 instance.

```plaintext
trivy --version
```

## **Step 4: Install Plugins like JDK, Sonarqube Scanner, NodeJs, and OWASP Dependency Check**

### **4A — Install Plugin**

Goto Manage **Jenkins** →**Plugins** → **Available Plugins**

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1702184807613/2f98168e-8595-4e9c-9ccc-04f5c18894dd.png align="center")

**Install below plugins**

1: Eclipse Temurin Installer (Install without restart)

2: SonarQube Scanner (Install without restart)

3: Sonar Quality Gates (Install Without restart)

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1702184893538/5a8f99d9-1d9b-4529-b19b-3377e9ae61c0.png align="center")

### **4B: Configure Java and Nodejs in Global Tool Configuration**

Goto Manage **Jenkins** → **Tools** → Install **JDK (17)** and **NodeJs (16)**. Click on **Apply** and **Save**

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1702185224674/852f608f-54b8-4d82-8f74-766101241a31.png align="center")

Choose the option **install from adoptium.net**

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1702185063580/151a5503-3fc4-4a49-93a2-7e94ee7ae46b.png align="center")

## **Step 5: Configure Sonar Server in Manage Jenkins**

Grab the public IP address of your EC2 instance.

Sonarqube works on Port 9000, so &lt;Public IP&gt;:9000.

Go to your Sonarqube server.

Click on **Administration** → **Security** → **Users** → Click on **Tokens** and **Update Token**, → Give it a name, and click on **Generate Token**

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1702185366020/fe4e754a-e0ad-4317-99e8-50884fa07296.png align="center")

click on update Token

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1694160866134/49f34dd2-de41-455c-aee0-f1ffe13d8488.png?auto=compress,format&format=webp&auto=compress,format&format=webp&auto=compress,format&format=webp&auto=compress,format&format=webp align="left")

Create a token with a name and generate

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1702185530058/331fbb68-f38f-4576-99e7-a41c98e9ed71.png align="center")

copy Token

Goto **Jenkins Dashboard** → **Manage Jenkins** → **Credentials** → Add **secret text**. It should look like this

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1702186101219/307a7eac-9b0f-47f1-a7c1-3379c21ae0b0.png align="center")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1694161147409/35b423f8-e7e4-402e-895b-cd1869b8a170.png?auto=compress,format&format=webp&auto=compress,format&format=webp&auto=compress,format&format=webp&auto=compress,format&format=webp align="left")

You will see this page once you click on create

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1694161221410/9a7a6f3f-fd40-4bbb-8857-bf2ffb8e6895.png?auto=compress,format&format=webp&auto=compress,format&format=webp&auto=compress,format&format=webp&auto=compress,format&format=webp align="left")

Now, go to Dashboard → **Manage Jenkins** → **System** and add like the below image.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1702185941846/21f3a1bb-b082-4266-b77f-ce8f21ce43cb.png align="center")

Click on Apply and Save.

**The Configure System option** is used in Jenkins to configure different server

**Global Tool Configuration** is used to configure different tools that we install using Plugins

We will install a sonar scanner in the tools.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1702186043630/afe376eb-6242-4f12-ac73-2bc744c05f98.png align="center")

In the Sonarqube Dashboard, add a quality gate as well.

In the sonar interface, create the quality gate as shown below:

Click on the **quality gate, then create.**

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1702186397756/2fb60e8b-4ceb-433b-bcf7-7187a2b26f90.png align="center")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1702186456129/748c72ea-42d8-48dc-a398-e8354df72917.png align="center")

Click on the **save** option.

In the Sonarqube Dashboard, Create **Webhook** option as shown in below:

**Administration--&gt; Configuration--&gt;Webhooks**

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1702186163006/74900198-c406-4564-b7d6-33daaf8a81e0.png align="center")

Click on **Create**

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1702186192291/45a3aa58-701c-423d-99b5-78c6087e57b6.png align="center")

Add details:

```plaintext
<http://jenkins-private-ip:8080>/sonarqube-webhook/
```

Let's go to our pipeline and add the script to our pipeline script.

```plaintext
pipeline{
    agent any
    tools{
        jdk 'jdk17'
        nodejs 'node16'
    }
    environment {
        SCANNER_HOME=tool 'sonar-scanner'
    }
    stages {
        stage('clean workspace'){
            steps{
                cleanWs()
            }
        }
        stage('Checkout from Git'){
            steps{
                git branch: 'main', url: 'https://github.com/sridhar-modalavalasa/Swiggy-Clone-App.git'
            }
        }
        stage("Sonarqube Analysis "){
            steps{
                withSonarQubeEnv('sonar-server') {
                    sh ''' $SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=Swiggy-CICD \
                    -Dsonar.projectKey=Swiggy-CICD '''
                }
            }
        }
        stage("quality gate"){
           steps {
                script {
                    waitForQualityGate abortPipeline: false, credentialsId: 'Sonar-token' 
                }
            } 
        }
        stage('Install Dependencies') {
            steps {
                sh "npm install"
            }
        }
}
}
```

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1702186731638/d28aeef4-b799-4982-9d47-86106a22bf1a.png align="center")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1702186789956/87e5ae26-98c0-4eed-8c2a-8e1444bda0e8.png align="center")

Click on Build now, and you will see the stage view like this:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1702186827482/f7064852-042b-4296-bb18-6826c1e6c674.png align="center")

To see the report, you can go to Sonarqube Server and go to Projects.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1702186853794/cc3ee1e6-fd9f-4d35-8208-d18abcb7290c.png align="center")

You can see the report has been generated, and the status shows as passed. You can see that there are 801 lines it has scanned. To see a detailed report, you can go to issues.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1702186906872/81b8db68-db95-491a-81d4-8f969a850a07.png align="center")

### **Step 6: Install OWASP Dependency Check Plugins**

Go to **Dashboard** → **Manage Jenkins** → **Plugins** → **OWASP Dependency-Check**. Click on it and install it without restarting.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1694162570631/a0da6454-e85a-4e27-908a-255b024bf834.png?auto=compress,format&format=webp&auto=compress,format&format=webp&auto=compress,format&format=webp&auto=compress,format&format=webp align="left")

First, we configured the plugin, and next, we had to configure the Tool

Goto **Dashboard** → **Manage Jenkins** → **Tools** →

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1702187136757/19f8c6e8-a1d2-4eba-b9c0-d109d7b04015.png align="center")

Click on **Apply** and **save** here.

Now go to **Configure** → Pipeline and add this stage to your pipeline and build.

```plaintext
stage('OWASP FS SCAN') {
            steps {
                dependencyCheck additionalArguments: '--scan ./ --disableYarnAudit --disableNodeAudit', odcInstallation: 'DP-Check'
                dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
            }
        }
        stage('TRIVY FS SCAN') {
            steps {
                sh "trivy fs . > trivyfs.txt"
            }
        }
```

You will see that in status, a graph will also be generated for vulnerabilities.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1698225711345/5765f1cb-fd23-416b-a25b-2157083bd213.png?auto=compress,format&format=webp align="left")

## **Step 7: Docker Image Build and Push**

We need to install the Docker tool on our system.

Go to **Dashboard** → **Manage Plugins** → **Available plugins** → Search for **Docker** and install these plugins.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1702187503440/4e287534-ef71-40c5-b9d0-0a40bcd3978e.png align="center")

Now, goto **Dashboard** → **Manage Jenkins** → **Tools** →

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1702187546212/135dee74-244f-4049-b226-44ed20eb827f.png align="center")

Now go to the Dockerhub repository to generate a token and integrate with Jenkins to push the image to the specific repository.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1702187650285/87ffeda0-8f6f-4bc1-a3a6-ec2ba593ca25.png align="center")

If you observe, there is no repository related to Swiggy.

There is an icon with the first letter of your name.

Click on that **My Account,** --&gt; **Settings --&gt; Create a new token** and copy the token.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1702187861896/cdcbfb78-f2c0-42bb-8112-2306fe2f1bd0.png align="center")

Goto **Jenkins Dashboard** → **Manage Jenkins** → **Credentials** → Add **secret text**. It should look like this:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1702187999216/9c11946b-5457-4463-88dc-92eb502a3c9c.png align="center")

Add this stage to Pipeline Script.

```plaintext
stage("Docker Build & Push"){
            steps{
                script{
                   withDockerRegistry(credentialsId: 'dockerhub', toolName: 'docker'){ 
                    app.push("${env.BUILD_NUMBER}")  
                       sh "docker build -t swiggy-app ."
                       sh "docker tag swiggy-app shreedhar4037/swiggy-app:latest "
                       sh "docker push shreedhar4037/swiggy-app:latest "
                    }
                }
            }
        }
        stage("TRIVY"){
            steps{
                sh "trivy image shreedhar4037/swiggy-app:latest > trivyimage.txt" 
            }
        }
```

You will be able to view the output in the Jenkins pipeline and output upon successful execution.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1702188219633/3e578e1c-e3bd-4e28-a010-10c18f2c5e9d.png align="center")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1702188231102/d2066445-9db3-4a36-8481-6056133b7c66.png align="center")

When you log in to Dockerhub, you will see a new image is created.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1702188267607/a7cd02d4-f8e4-4532-b044-c123bf8f56ab.png align="center")

## **Step 8: Creation of EKS Cluster with ArgoCD**

**EKSCTL Installation:**

**Now let's install EKSCTL in Ubuntu EC2 which was created earlier.**

```plaintext
ARCH=amd64
PLATFORM=$(uname -s)_$ARCH
curl -sLO "https://github.com/weaveworks/eksctl/releases/latest/download/eksctl_$PLATFORM.tar.gz"
tar -xzf eksctl_$PLATFORM.tar.gz -C /tmp && rm eksctl_$PLATFORM.tar.gz
sudo mv /tmp/eksctl /usr/local/bin

# Check the eksctl version.
eksctl version
```

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1702188593459/b6f45db2-435e-4803-8fc2-161ddbf22faa.png align="center")

**Command to Create EKS Cluster using eksctl command:**

```plaintext
eksctl create cluster --name <name-of-cluster> --nodegroup-name <nodegrpname> --node-type <instance-type> --nodes <no-of-nodes>

eksctl create cluster --name my-eks-cluster --nodegroup-name ng-test --node-type t3.medium --nodes 2
```

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1702188640835/19b0d9b0-1012-4758-ad9f-76803598cfeb.png align="center")

**It will take 5–10 minutes to create a cluster.**

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1702188680866/d6ea5c52-1890-4006-9a0c-1dd9de03a635.png align="center")

As you will see in the EC2 instances running list one instance is running in the name of EKS Cluster as shown below.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1702188709887/3e4e4ef6-4995-46f2-9df6-8825d7c82c3f.png align="center")

**EKS Cluster is up and ready and check with the below command.**

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1702188737441/0946e876-104e-44eb-ae8b-815e34115073.png align="center")

**Now let's install ArgoCD in the EKS Cluster.**

```plaintext
kubectl create ns Argocd
# This will create a new namespace, argocd, where Argo CD services and application resources will live.
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1702189008925/f8ba1a3f-e38d-446b-9d07-211cf135ec8a.png align="center")

**Download Argo CD CLI:**

```plaintext
curl -sSL -o argocd-linux-amd64 https://github.com/argoproj/argo-cd/releases/latest/download/argocd-linux-amd64
sudo install -m 555 argocd-linux-amd64 /usr/local/bin/argocd
```

**Access The Argo CD API Server:**

```plaintext
# By default, the Argo CD API server is not exposed with an external IP. To access the API server, 
choose one of the following techniques to expose the Argo CD API server:
* Service Type Load Balancer
* Port Forwarding
```

**Let's go with Service Type Load Balancer.**

```plaintext
# Change the argocd-server service type to LoadBalancer.
kubectl patch svc argocd-server -n argocd -p '{"spec": {"type": "LoadBalancer"}}'
```

**List the resources in the namespace:**

```plaintext
kubectl get all -n argocd
```

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1702189029575/345a9160-9e48-4fd8-a31c-9524689ac37e.png align="center")

**Get the load balancer URL:**

```plaintext
kubectl get svc -n argocd
```

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1702189139309/595c42c7-eb10-4017-b826-1e3be30d75b2.png align="center")

Pickup the URL and paste it into the web to get the UI as shown below image:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1702189158105/55accaa0-66e9-4a1e-90bd-ce260a0f561c.png align="center")

**Login Using The CLI:**

```plaintext
argocd admin initial-password -n argocd
```

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1702189188690/fd60f8cc-9f06-4ca7-b5f6-5c98ac5d6e48.png align="center")

Login with the admin and Password in the above you will get an interface as shown below:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1702189214666/11a907bb-59b7-43b3-9458-c411fe842c2b.png align="center")

**Click on New App:**

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1702189254764/1ba774e4-a32f-48aa-b929-04532c28054f.png align="center")

**Enter the Repository URL, set path to ./, Cluster URL to** [**kubernetes.default.svc**](https://kubernetes.default.svc/)**, the namespace to default and click save.**

The GitHub URL is the Kubernetes Manifest files which I have stored and the pushed image is used in the Kubernetes deployment files.

**Repo Link:** [https://github.com/sridhar-modalavalasa/Swiggy-App-ArgoCD](https://github.com/sridhar-modalavalasa/Swiggy-App-ArgoCD)

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1702189430681/316a7c00-4492-4d00-98de-9def0134d52d.png align="center")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1702189297343/b0226e35-8899-4794-b405-7b37dcc55347.png align="center")

**You should see the below, once you're done with the details.**

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1702189468610/4dcf7081-fee3-4a13-be28-e7854511613a.png align="center")

**Click on it.**

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1702189494035/b7f5de47-3b66-4079-b099-f87b86caf2f0.png align="center")

**You can see the pods running in the EKS Cluster.**

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1702189526068/feba6e26-581c-46c9-ad81-801a31aa9077.png align="center")

**We can see the out-of-pods using the load balancer URL:**

```plaintext
kubectl get svc
```

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1702189546499/4b8b1b9d-5bd4-4124-8331-8c0e0ab1e1e7.png align="center")

With the above load balancer, you will be able to see the output as shown in the below image:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1702189702622/6ee87f27-bfe2-4484-9e58-e1b5950b9b6a.png align="center")

As you observe in the above image, I just want to change the address of the Swiggy Application.

Then the real magic will happen, the changes updated in the code we will try to push the code to GitHub and run a pipeline to push the image to the repository with the updated details.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1702189918059/6594a8d8-76e9-4874-8517-4fb15666740f.png align="center")

I will click on the sync option which is in the ArgoCD and then the updates will be in our Swiggy Website.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1702189975536/da5ef9e8-b5e5-44f6-8dd5-e919d1b0bfbe.png align="center")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1702189986080/43c8682e-7e24-428c-ba96-549d35b2f8e9.png align="center")

I have changed the website details of the address in the code and then you can see the results in the below image.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1702190030121/33e6d0c7-2f4c-446b-a760-925c9ac34e42.png align="center")

## Conclusion:

In this project, we have covered the essential steps to deploy a Swiggy app with a strong focus on security through a DevSecOps approach. By leveraging tools like Terraform, Jenkins CI/CD, SonarQube, Trivy, Argocd, and EKS, we can create a robust and secure pipeline for deploying applications on AWS.

Remember that security is an ongoing process, and it is crucial to stay updated with the latest security practices and continuously monitor and improve the security of your applications. With the knowledge gained from this guide, you can enhance your DevSecOps skills and ensure the smooth and efficient deployment of secure applications on Amazon Web Services.

**Thank you for reading this post! I hope you found it helpful. If you have any feedback or questions, Please connect with me on LinkedIn at**

[https://www.linkedin.com/in/sridhar-modalavalasa-12b492173/](https://www.linkedin.com/in/sridhar-modalavalasa-12b492173/)

[https://github.com/sridhar-modalavalasa](https://github.com/sridhar-modalavalasa)

**Your feedback is valuable to me. Thank you!**