---
title: "Part 2: Implementation of CI/CD on Kubernetes using Argo CD."
datePublished: Sun Jun 25 2023 11:12:39 GMT+0000 (Coordinated Universal Time)
cuid: cljbbz9p70bpf39nvercx39p3
slug: part-2-implementation-of-cicd-on-kubernetes-using-argo-cd
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1687675098695/a0e5dece-0a51-45bf-9cfd-a14d88d921d1.png
tags: cicd-on-kubernetes-using-argocd

---

Let's continue where we have left our process in the previous post. In this, you will gonna experience the whole application deployment using CI/CD on Kubernetes using ArgoCD.

I will walk you through every process that I have done in my Flask application deployment.

### Prerequisites:

* **Jenkins Server up and running.**
    
* **Docker and git installed inside Jenkins Server.**
    
* **Docker Hub account.**
    
* **AWS Account.**
    
* **GitHub Account.**
    
* **EKS Cluster running**
    
* **Basic Understanding of Jenkins, Docker and Kubernetes.**
    

### Jenkins Server Up and Running:

I have installed my Jenkins in the AWS EC2 instance with the following details which are mentioned below:

* **t2.micro**
    
* **security groups are arranged in this way before the package installation for the Jenkins server.**
    

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1687675990384/f4188c92-1fd3-4f7d-86a8-5b102143e0ad.png align="center")

* After launching the EC2 instance you need to follow the below documentation link which is the official documentation of Jenkins for Amazon Linux2.
    
* [Jenkins installation](https://www.jenkins.io/doc/tutorials/tutorial-for-installing-jenkins-on-AWS/)
    
* After installing Jenkins try to checkout with the IP address of the machine with port 8080. Then you can check the web as shown in the below image.
    

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1687676239582/70701af5-9104-497a-b481-0928b75fcc46.png align="center")

Then our next requirement is to set up the docker in the Jenkins server for running the jobs which we are assigned to the pipeline.

### **Docker and git installed inside Jenkins Server:**

Now let's download the Docker on the same server.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1687676503561/dffb057e-e9fb-4c06-b66e-36067e1394ee.png align="center")

```plaintext
# Become root user
sudo su -

# Apply updates
yum update -y

# Install Docker
yum install docker -y

#sudo systemctl status docker
sudo systemctl start docker.service -->   To start the service
sudo systemctl stop docker.service -->    To stop the service
sudo systemctl restart docker.service --> To restart the service
sudo systemctl status docker.service -->  To get the service status
```

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1687676649189/ecdf1e78-5089-4d40-9dd7-e1f5557005c4.png align="center")

Now both Jenkins and Docker are installed on our server.

**Install Git also on the same server.**

Follow the below steps for the git installation in the same Jenkins server.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1687676772729/4b3fed45-c71d-42f0-b08f-d7a0bfc0d29e.png align="center")

```plaintext
# Install git
yum install git -y
 
# Check the version of git
git --version
```

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1687676847487/245c89e1-b9f6-422a-b9bc-0159caf00cbd.png align="center")

**Integrate Docker with Jenkins.**

You have to run docker commands using jenkins user, while running jenkins job.

```plaintext
# Add Jenkins user to Docker Group.
usermod -a -G docker jenkins

# Reload a Linux user's group assignments to docker
newgrp docker

# Check the user id. to see group we added 
id jenkins
```

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1687677102556/451479ef-f45f-44ef-b4e6-7cbc06604718.png align="center")

After the successful completion of this setup, we have to install the plugins in Jenkins which is related to the docker. Because we are going to run the Jenkins pipeline for successful running of jobs we need to install the plugins.

Follow the below steps for installing the plugins in Jenkins.

In the **Jenkins dashboard -&gt; Manage plugins -&gt; Search for Docker pipeline -&gt;**

**Install without restart**

as shown in the below images.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1687677617141/7c0b69d2-4b6b-4f55-a307-fcc6856eb126.png align="center")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1687677635670/0e4d96f5-5778-4fea-ad96-10833b990eec.png align="center")

Try to install this plugin and after that follow the below steps for further process.

### Create a Repository name "flask-app" in your Docker Hub.

If you are not having a docker hub account try to create an account because this was the account we will use for push and pull images and then we will use that image in Kubernetes manifest file.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1687678032140/b067da83-b427-41b6-9aaf-9a984158a3c9.png align="center")

**Add your GitHub and Docker Hub credentials to your Jenkins credentials.**

For the above process, you need GitHub and Dockerhub credentials. With these credentials, we are gonna decide where the image has to be stored and use for our requirements.

For GitHub credentials follow the below steps:

* Go to your **GitHub account -&gt; settings -&gt; Developer settings -&gt; Personal Access Tokens -&gt; Token classic -&gt;Generate new token.**
    

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1687678473006/71d1c8de-4f48-4cfd-9b23-7c41c5336817.png align="center")

Now you will get doubt why we have created this token. Now we will add this **token** and **GitHub username** to the Jenkins **Dashboard -&gt; Mange Jenkins -&gt; Credentials -&gt; System -&gt; Global credentials.**

In the same way, add your **Dockerhub username and password** in the above path in the Jenkins dashboard and save it.

After saving the credentials you will get an interface like this which is shown in below:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1687678818608/c6230a38-66ff-423d-8178-dd7d23e6385b.png align="center")

* **Save GitHub credentials with ID "github"**
    
* **Save Docker Hub credentials with ID "dockerhub"**
    
* **Be specific with the "ID's" because these are the things we will use in the Jenkins pipeline as variables.**
    

Now we have done with the Docker, Git and Jenkins complete setup and we can run our pipeline to push the image to the Dockerhub that we have prepared.

### **Dockerfile and Jenkinsfile walkthrough:**

let's go over the GitHub repositories first this is the application code repository where we have our application code**(app.py), Dockerfile, Jenkinsfile, and requirements.**

[Flask-code GitHub Repo](https://github.com/sridhar-modalavalasa/K8sArgoCD-flask-code)

```plaintext
app.py 
from flask import Flask
app = Flask(__name__)

@app.route('/')
def hello_world():
    return '<h1>Hello Docker</h1>'
```

* it is a very simple Python **app.py** program which is importing the library flask and just returning hello docker the file.
    
* The **requirements.txt** lists the external library flask and in this case, we are specifically using version 2.2.0.
    
* The **Docker file** dockerizes that python program and creates a container image it is using the base python 3.9 docker image and then it's copying over the requirement file running a pip install of the flask then it is running the python program accepting incoming connections.
    

```plaintext
# syntax=docker/dockerfile:1

FROM python:3.9.16-slim-buster

WORKDIR /myapp

COPY requirements.txt requirements.txt
RUN pip3 install -r requirements.txt

COPY . .

CMD [ "python3", "-m" , "flask", "run", "--host=0.0.0.0"]
```

Now the Jenkinsfile is for the job which is creating the docker container image.

so the **first stage** clones this repository into the Jenkins environment.

```plaintext
stage('Clone Repository') {
      

        checkout scm
    }
```

 The **second** stage builds the docker container image so basically it looks for a docker file in the repository that it just cloned and then runs that docker file. The **shreedhar4037/flask-app.**

Where **shreedhar4037** is the docker hub account id where **flask-app** is the repository.

```plaintext
stage('Build Image') {
  
       app = docker.build("shreedhar4037/flask-app")
    }
```

In the **third stage,** you can run some unit testing.

```plaintext
stage('Test Image') {
  

        app.inside {
            sh 'echo "Tests passed"'
        }
    }
```

In the **fourth stage** where we push the image to the docker hub so you give the URL of the official docker hub website and in the next part docker hub is not a standard keyword.

If you have remembered the docker hub is the id we have mentioned in the credentials in the Jenkins in this particular path copy and paste the id in the pipeline.

**Dashboard -&gt; Mange Jenkins -&gt; Credentials -&gt; System -&gt; Global credentials.**

when we set up our Jenkins environment and push this container image with the tag of the build number of this job. so as you keep pushing more code changes these Jenkins jobs will keep resubmitting themselves with the new build number that will be used as the tag for the new container images.

```plaintext
stage('Push Image') {
        
        docker.withRegistry('https://registry.hub.docker.com', 'dockerhub') {
            app.push("${env.BUILD_NUMBER}")
        }
    }
```

**Note:** This **env.BUILD\_NUMBER** is a standard variable that is available readily in any Jenkins job. you don't need to define this variable anywhere.

In the fifth stage we trigger another Jenkins job to update the deployment file and the name of this job is **Updating-manifest**.

we are updating the deployment file with the build number from this job.

so we have to pass this build number and we are passing that build number into a parameter called docker tag to this job.

```plaintext
stage('Trigger ManifestUpdate') {
                echo "triggering updatemanifestjob"
                build job: 'Updating-manifest', parameters: [string(name: 'DOCKERTAG', value: env.BUILD_NUMBER)]
        }
}
```

we are going to see that in a minute this is the **K8s-Manifests-ArgoCD** repository. so this repository has the deployment file and Jenkins file for the jenkins job to update the deployment.

[K8sManifests-repo with Jenkins file](https://github.com/sridhar-modalavalasa/K8s-Manifests-ArgoCD)

so if we go to the **deployment. yaml** the container image is referencing to the latest tag. we run our Jenkies job it is going to update this tag and in the next step we are creating a load balancer service to talk to the container by default flask application runs on the target port 5000 in the container so that's why the URL of the load balancer will be using port 80 for you but it is redirecting the traffic to the port 5000 of the container.

```plaintext
apiVersion: apps/v1
kind: Deployment
metadata:
  creationTimestamp: null
  labels:
    app: myapp
  name: myapp
spec:
  replicas: 3
  selector:
    matchLabels:
      app: myapp
  strategy: {}
  template:
    metadata:
      creationTimestamp: null
      labels:
        app: myapp
    spec:
      containers:
      - image: shreedhar4037/flask-app:7
        name: myapp
        resources: {}
status: {}
---
apiVersion: v1
kind: Service
metadata:
  name: lb-service
  labels:
    app: lb-service
spec:
  type: LoadBalancer
  ports:
  - port: 80
    targetPort: 5000
  selector:
    app: myapp
```

I have opened the Jenkins file for updating the deployment. So the **first stage** is very similar it clones this repository in the Jenkins environment.

```plaintext
stage('Clone repository') {
      

        checkout scm
    }
```

In the **second stage,**

* It updates the file so it sets the git config user email and username for the commit history.
    
* I'm running edit cat deployment.yaml to show you how it is before we change.
    
* Then we run this sed command to change this container image this sed command is using the symbol plus as the delimiter because we cannot use the default front slash because our container image has a front slash so sed won't work.
    
* So **sed -I** mean it is going to do an in-place replacement in the file itself instead of creating a new file.
    
* Then I am looking for the **shreedhar4037/flask-app.\*** means anything afterward in the line is fine.
    
* I am replacing the line with **shreedhar4037/flask-app:${DOCKERTAG}** remember docker tag is the parameter that this Jenkins job will accept as input from the other Jenkins job**.**
    
* I'm doing this change on the deployment.yaml file then I'm displaying the deployment file again to show you.
    
* I'm running git add and git commit and git push.
    
* So now let's go over the first line in the first line note that this credential id GitHub is fetching the credentials saved under the id GitHub from the Jenkins
    
    credentials.
    
* If I switch to my Jenkins you will see the id **github** has my GitHub credentials remember that the password of the GitHub should be your token and not the password.
    
* so we'll go over it when we set up Jenkins and after it fetches that username and password it runs the git push command for this Kubernetes manifest repo and updates the deployment file alright so now let's jump into Jenkins and set up the jobs.
    

```plaintext
 stage('Update GIT') {
            script {
                catchError(buildResult: 'SUCCESS', stageResult: 'FAILURE') {
                    withCredentials([usernamePassword(credentialsId: 'github', passwordVariable: 'GIT_PASSWORD', usernameVariable: 'GIT_USERNAME')]) {
                      
                        sh "git config user.email sridhar02101998@gmail.com"
                        sh "git config user.name sridhar-modalavalasa"
                      
                        sh "cat deployment.yaml"
                        sh "sed -i 's+shreedhar4037/flask-app.*+shreedhar4037/flask-app:${DOCKERTAG}+g' deployment.yaml"
                        sh "cat deployment.yaml"
                        sh "git add ."
                        sh "git commit -m 'By Jenkins Job changemanifest: ${env.BUILD_NUMBER}'"
                        sh "git push https://${GIT_USERNAME}:${GIT_PASSWORD}@github.com/${GIT_USERNAME}/K8s-Manifests-ArgoCD.git HEAD:main" 
      }
    }
  }
}
```

**After forking Repositories make these changes immediately.**

```plaintext
In python-flask code repo /K8sArgoCD-flask-code
replace "shreedhar4037/flask-app" with "<your-dockerhub-username>/<repo-name>"
Note:- better don't change your repo name in docker hub.
```

```plaintext
In K8s-Manifests-ArgoCD code repo make these changes in below 
sh "git config user.email sridhar02101998@gmail.com" ---> sh "git config user.email <your-email-id>" 
sh "git config user.name sridhar-modalavalasa"        ---> sh "git config user.name <git-user-name>"

sh "sed -i 's+shreedhar4037/flask-app.*+shreedhar4037/flask-app:${DOCKERTAG}+g' deployment.yaml"
sh "sed -i 's+<your-docker-hub-usename>/packages.*+<your-docker-hub-usename>/packages:${DOCKERTAG}+g' deployment.yaml"
```

```plaintext
In K8s-Manifests-ArgoCD/deployment.yaml
replace
- image: shreedhar4037/flask-app:5  ---> - image: <your-docker-hub-username>/flask-app:5
```

### **Jenkins job setup:**

**Let's create two jobs on Jenkins.**

### JOB1:

To create a new jenkies job click the new item give the name Building-image and then select pipeline click ok and scroll down to pipeline.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1687686881975/52ce73e4-e6fe-4da0-8e61-f60e60bc962a.png align="center")

Instead of giving the pipeline script here, we want to grab it from GitHub so select pipeline script from scm and then git under sem go back to GitHub [Flask-app code repo](https://github.com/sridhar-modalavalasa/K8sArgoCD-flask-code) repository click code copy the https URL and then paste it in the repository URL as shown in below.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1687687096297/11ec9583-51d8-4e4b-805e-aeb15100f68a.png align="center")

If you scroll down a little bit there will be an option as shown in the below image. Give your branch name. Save the pipeline.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1687687172246/a97d15c4-6c77-4696-af05-15291a44d423.png align="center")

### JOB2:

Now let's set up the **Updating-manifest** Jenkins job go back to the dashboard new item and give the name update manifest note that this name should match whatever you use in the Jenkins file for the build container image job going back to Jenkins.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1687687296783/371028b3-a7b5-49c3-8beb-5864adc75e55.png align="center")

This is also a pipeline job click ok then select this project is parameterized and then add a string parameter and the name of the parameter is docker tag the default value is the latest but we are going to override it from the previous job and then scroll down the same thing here we are going to grab the Jenkins file from the git and in this case, you have to give the URL for the Kubernetes manifest. The repo URL is down below:

[K8s-Manifest-repo url](https://github.com/sridhar-modalavalasa/K8s-Manifests-ArgoCD)

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1687687370698/bcbe95ae-cd70-429d-afdb-c803f46e1b5f.png align="center")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1687687488419/c9eee0f6-4d2d-473b-bcbe-2c5831bfc0e3.png align="center")

so now let's try to run this go to the Build image job and click **Building-image** now while we wait for that job to complete and then it will trigger the **Updating-manifest** job as shown in the below.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1687687662286/e499f21f-fc89-46d3-b8b2-7616941a3614.png align="center")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1687687693580/16b11203-d50d-4815-b7c9-38fcf65e84ea.png align="center")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1687687703704/bb55519e-9623-42b1-8387-1360543d450d.png align="center")

The Two jobs are triggered successfully.

**Congratulations you have done 90 % of the Project as of now. Now we just need to set up EKS Cluster and install ArgoCD init.**

**Launch a new instance and install AWS CLI, eksctl, and kubectl into Create, and interact with EKS Cluster in AWS.**

* Select Amazon Linux-2 AMI
    
* .t2.micro
    
* AWS CLI Installation
    

```plaintext
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
unzip awscliv2.zip
sudo ./aws/install

# Check AWS CLI version
aws --version
```

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1687687957939/c47241d1-562b-46ca-99b0-a025aa7deaac.png align="center")

**Now Configure the AWS CLI with your AWS "Access key" and "Secret access key".**

```plaintext
aws configure
```

**Now Check by listing s3 buckets in your AWS account using AWS CLI.**

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1687688027841/fce2dbf8-b06f-474c-b796-5707ea168f85.png align="center")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1687688044398/2480a569-92f8-423e-8641-482774c41263.png align="center")

### **EKSCTL Installation :**

**Now lets install eksctl.**

```plaintext
ARCH=amd64
PLATFORM=$(uname -s)_$ARCH
curl -sLO "https://github.com/weaveworks/eksctl/releases/latest/download/eksctl_$PLATFORM.tar.gz"
tar -xzf eksctl_$PLATFORM.tar.gz -C /tmp && rm eksctl_$PLATFORM.tar.gz
sudo mv /tmp/eksctl /usr/local/bin

# Check the eksctl version.
eksctl version
```

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1687688186449/5592cd0f-cc65-43a9-b1c8-684e3952dcd6.png align="center")

**Let's Install kubectl:**

```plaintext
curl -O https://s3.us-west-2.amazonaws.com/amazon-eks/1.27.1/2023-04-19/bin/linux/amd64/kubectl
chmod +x ./kubectl
mkdir -p $HOME/bin && cp ./kubectl $HOME/bin/kubectl && export PATH=$HOME/bin:$PATH
echo 'export PATH=$HOME/bin:$PATH' >> ~/.bashrc
kubectl version --short --client
```

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1687688211148/d559d7aa-7d15-4d91-90d3-f807feeea0fd.png align="center")

**Command to Create EKS Cluster using eksctl command:**

```plaintext
eksctl create cluster --name <name-of-cluster> --nodegroup-name <nodegrpname> --node-type <instance-type> --nodes <no-of-nodes>

eksctl create cluster --name mycluster --nodegroup-name ng-test --node-type t3.medium --nodes 2
```

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1687688335808/e7ed507b-7b8b-442f-90fb-4281ecb6a8e4.png align="center")

**It will take 5-10 mins to create cluster.**

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1687688392572/1da47e9e-ce64-4d5a-b323-61071a4fac6b.png align="center")

**We can see as of now there is a cluster in EKS.**

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1687688421839/6b92c867-5cb4-4c88-acce-183814600bac.png align="center")

As you will see in the EC2 instances running list two instances are running in the name of EKS Cluster as shown below.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1687688514506/183b20fe-c601-4e39-bf39-5587163beb8b.png align="center")

**EKS Cluster is up and ready.**

**Now let's install ArgoCD in EKS Cluster.**

```plaintext
# This will create a new namespace, argocd, where Argo CD services and application resources will live.
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```

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

**Get the load balancer URL:**

```plaintext
kubectl get svc -n argocd
```

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1687689051281/318ad320-b78f-4d74-950c-ece6aeb90cf6.png align="center")

Pickup the URL and paste it in the web for getting the UI as shown below image:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1687689112730/09e784d9-b713-4044-aa98-724e3d5a7b00.png align="center")

**Login Using The CLI:**

```plaintext
argocd admin initial-password -n argocd
```

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1687689169558/d810b248-86ae-4296-9e15-53580c08cf8c.png align="center")

Login with the admin and Password in the above you will get an interface as shown below:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1687689203203/16e76bcf-7df9-4174-bd75-ea0bf3fc6a07.png align="center")

**Click on New App:**

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1687689287583/b3668b8a-d84d-40f4-951d-74609562662c.png align="center")

**Enter the Repository URL, set path to ./, Cluster URL to** [**https://kubernetes.default.svc**](https://kubernetes.default.svc)**, the namespace to default and click save.**

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1687689356955/1437fbd5-546e-40cc-b4ff-b35ff8e656ca.png align="center")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1687689405754/4dda50f5-35d8-4e3b-bef3-118730a34c8e.png align="center")

**You should see the below, once you're done with the details.**

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1687689438478/56502e0a-0471-455a-9320-085c387ea475.png align="center")

**Click on it.**

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1687689467383/a1f998b5-c0bd-4bd0-907b-8460f0d133c0.png align="center")

**You can see the pods running in EKS Cluster.**

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1687689525781/54332090-26c1-42b9-9b68-eca82d652a35.png align="center")

**We can see the out-of-pods using the load balancer URL:**

```plaintext
kubectl get svc
```

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1687689891637/da1a3a87-4def-41fb-a10f-ae56988baef9.png align="center")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1687689909812/fcf3e307-979d-4725-bd1c-bdb9a05ef742.png align="center")

**ArgoCD will automatically syn every 3 mins to the manifest repo to pull and apply changes to EKS Cluster.**

**If you are interested you can apply the GitHub webhook to automatically trigger the Jenkins job when developers commit changes in a git repo. So that ArgoCD can pull those changes and apply them in EKS Cluster.**

**Let's Do some code changes and the output will automatically change or not.**

Let's go and trigger the jobs then it will automatically trigger the ArgoCD due to the code change in the repository.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1687690222182/4fe332f3-085b-4a00-9aba-578036033b6a.png align="center")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1687690463450/93213c86-5861-4a72-9b5c-993cb7a04b54.png align="center")

The Building-image is triggered and the job is created and then the next triggering job is the Updating-manifest.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1687690247055/1c5ab794-97ba-4663-833a-3374612c89e1.png align="center")

The ArgoCD observed some changes and it is updating in the pods as well. Now let's go and check the load balancer of the service.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1687690338637/d9478e05-87df-4386-9110-825702252c1b.png align="center")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1687690427360/d60fc15d-26fb-4036-b0dc-a824740dd301.png align="center")

**We have successfully deployed an Application to an EKS Cluster using Jenkins and ArgoCD.**

**Thank you for reading this post! I hope you found it helpful. If you have any feedback or questions, Please connect with me on LinkedIn at**

[https://www.linkedin.com/in/sridhar-modalavalasa-12b492173/](https://www.linkedin.com/in/sridhar-modalavalasa-12b492173/)

[https://github.com/sridhar-modalavalasa](https://github.com/sridhar-modalavalasa)

**Your feedback is valuable to me. Thank you!**

### #10WeeksOfCloudOps.

### References:

[https://kubernetes.io/](https://kubernetes.io/)

[https://argo-cd.readthedocs.io/en/stable/](https://argo-cd.readthedocs.io/en/stable/)

[https://www.youtube.com/watch?v=eqiqQN1CCmM](https://www.youtube.com/watch?v=eqiqQN1CCmM)