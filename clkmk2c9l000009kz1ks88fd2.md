---
title: "Automating Nginx Installation and Application Deployment Across Multiple Servers using Ansible & Jenkins."
datePublished: Fri Jul 28 2023 12:24:10 GMT+0000 (Coordinated Universal Time)
cuid: clkmk2c9l000009kz1ks88fd2
slug: automating-nginx-installation-and-application-deployment-across-multiple-servers-using-ansible-jenkins
tags: ansible-and-jenkins-automation

---

In this blog post, we dive into the world of Ansible projects, exploring how to structure and organize your automation workflows effectively.

Discover best practices for creating Ansible projects that simplify configuration management, application deployment, and system orchestration. Harness the power of Ansible to streamline your infrastructure management and drive efficiency in your IT operations.

The Git repository of the below project is pasted in below.

[Git Repository](https://github.com/sridhar-modalavalasa/Ansible-web-app)

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1690540958062/759cb747-5f80-4eae-9c72-5574ac9e8811.png align="center")

# **Deploy a Website 🌐using Ansible Playbook through Jenkins.**

Create 3 EC2 instances. Out of 3, One will be the Ansible Master node and the other 2 are child nodes. Make sure all two child nodes are created with the same key pair as Ansible Master.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1690538612196/dc809006-2fc2-4680-953f-812e62ca9bbf.png align="center")

Install Ansible on the Ansible Master node only.

```plaintext
# Installation of python 
$ apt install software-properties-common 
$ add-apt-repository ppa:deadsnakes/ppa 
$ apt install python3.11 
$ sudo update-alternatives --install /usr/bin/python python /usr/bin/python3.11 1

# Installation of Ansible 
$ sudo apt-add-repository ppa:ansible/ansible
$ sudo apt update
```

Once the installation is complete, you can check the version of Ansible using the following command:

```plaintext
$ ansible --version 
```

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1690539109797/bc45f3a7-2b0c-4e8b-829d-ea40cf15155b.png align="center")

Check the above command in the master node itself.

After completion of the installation. Now we have to make the connectivity between the Ansible master and the Worker machines.

## SSH Connectivity:

The easiest and safest connectivity between the servers. Now you have to follow the below process for making the connection between the servers.

Now we have to use one command for making the connection between the servers.

```plaintext
$ ssh-keygen
```

We have to generate a key in the three nodes as shown below images.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1690539618447/1d6e0a6f-6d22-4bdf-be8c-c29c5cfb2b83.png align="center")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1690539635210/5a448ce8-3f7a-4f29-a872-3a2392c808b4.png align="center")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1690539659997/c0e83e98-49d5-41dd-803f-c21e15184caa.png align="center")

In the master node, you have to redirect into the specific directory and copy the key of the master and paste it into the worker nodes as shown below.

### **Master Node:**

```plaintext
$ cd ~/.ssh/ 
$ cat cat id_rsa.pub 
```

After copying the above key you have to paste it to the remote servers as shown in the below process.

### **Worker Nodes:**

```plaintext
$ cd ~/.ssh/ 
$ vi authorized_keys 
```

Whatever the key you have copied in the above. You have to paste it into the remote server as shown in the above process.

After doing the process you will be able to access the server without the password as shown in the below image.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1690540144200/19d2cbdf-473d-4db0-898e-208ad223de55.png align="center")

Do the same process in the other worker node as well. Now we have to set up Jenkins in the **master server**.

## Jenkins Setup:

**Install the Jenkins server in the Ansible master**. By using these commands you will be able to access the Jenkins server.

```plaintext
# Installing the Jenkins Server. 
$ sudo apt update
$ sudo apt install default-jdk -y 
$ curl -fsSL https://pkg.jenkins.io/debian/jenkins.io-2023.key | sudo tee \
  /usr/share/keyrings/jenkins-keyring.asc > /dev/null 
$ echo deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc] \
  https://pkg.jenkins.io/debian binary/ | sudo tee \
  /etc/apt/sources.list.d/jenkins.list > /dev/null
$ sudo apt update 
$ sudo apt install jenkins -y 

# Installation of Git 
$ sudo apt install git
```

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1690540544211/230bc288-2443-4741-aefb-c1922600a1f4.png align="center")

After installing Jenkins. Now we have to integrate Jenkins with Ansible. For that, we need to follow the below process. First of all, we need to install the plugins which are required for the Ansible playbooks to run and execute in the background.

**Manage Jenkins** --&gt; **Plugins --&gt; Avaialable Plugins --&gt; Ansible**

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1690540837078/a0fb31e2-4ff2-456b-81b9-870eb5aaa5ee.png align="center")

Now we have to install the Ansible plugin as shown in the below image.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1690540924587/cb3085cc-0ab3-430a-a808-6bb296c5a519.png align="center")

Then we need to go to the **Manage Jenkins --&gt; Tools section.**

In that Tools section, If you scroll down a little bit. The Ansible tool will be available in that we need to ass the path of the Ansible master server. So that it would be able to run the Ansible commands as shown in the below image.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1690541163973/cf9ec696-efbd-4d91-bff1-080bd49be584.png align="center")

As you have seen in the above image. **I have mentioned the path of Ansible which is append running in the same server which is Jenkins running.**

Now we have to configure the playbooks and the pipelines for running the playbooks through Jenkins CI/CD.

Now we need to jump on to the host's file which is also called as Inventory file and add the list of hosts or servers, add the IP addresses of the servers.

```plaintext
[my-servers]
server1 ansible_host=172.31.80.149
server2 ansible_host=172.31.83.39

[all:vars]
ansible_python_interpreter=/usr/bin/python3
ansible_user=ubuntu
```

For copying the IP addresses of the server you have to copy the private Ip addresses of the server as shown in the below image and paste it into the host's file. So copy the other server details and paste it in the same manner.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1690541869788/ec080bb5-0230-44b0-a1d5-7626ffaf49e7.png align="center")

Now create a playbook to install Nginx. The sample web application will be deployed on top of the Nginx application.

```plaintext
- name: Configure nginx web server
  hosts: my-servers
  become: true
  become_user: root
  become_method: sudo
  tasks:
  - name: update apt
    apt:
      update_cache: yes
  - name: Install nginx
    apt:
      name: nginx
      state: latest
  - name: Start Nginx
    service:
      name: nginx
      state: started
      enabled: yes
  - name: Deploy web app
    copy:
      src: index.html
      dest: /var/www/html/
```

The code which is provided is an Ansible playbook, which is a collection of tasks that can be used to automate the configuration of systems. In this case, the playbook is used to configure a Nginx web server.

The become directive tells Ansible to run the tasks as the root user. This is necessary because some tasks, such as installing packages, require root privileges.

The **become\_user** and **become\_method** directives specify the user and method that Ansible will use to become root. In this case, the user is set to root and the method is set to sudo.

The hosts directive specifies the hosts that the playbook will run on. In this case, the hosts are set to **my-servers.**

### TASK 1: Update apt

The first task in the playbook updates the apt cache. This is necessary because Ansible uses apt to install packages, and the apt-cache needs to be up-to-date for Ansible to find the packages it needs.

### TASK 2: Install nginx

The second task in the playbook installs Nginx. The name parameter specifies the name of the package to install, and the state parameter specifies the state of the package after the installation is complete. In this case, the state is set to the latest, which means that Ansible will install the latest version of Nginx.

### TASK 3: Start Nginx

The third task in the playbook starts Nginx and enables it to start automatically at boot. The name parameter specifies the name of the service, and the state parameter specifies the state of the service after the task is complete. In this case, the state is set to start, which means that Ansible will start Nginx and the enabled parameter is set to yes, which means that Ansible will enable Nginx to start automatically at boot.

### TASK 4: Deploying a web app

We have created a sample HTML web application and we are trying to deploy the web application on top of the Nginx server. The below code is the sample web application.

```plaintext
<!DOCTYPE html>
<html>
<head>
    <title>Welcome Page</title>
    <style>
        body {
            font-family: Arial, sans-serif;
            background-color: bisque;
            margin: 0;
            padding: 0;
        }

        .container {
            max-width: 800px;
            margin: 100px auto;
            padding: 40px;
            text-align: center;
            background-color: beige;
            border-radius: 5px;
            box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
        }

        h1 {
            color: #333333;
            font-size: 36px;
            margin-bottom: 20px;
        }

        p {
            color: #222222; /* Updated the color to dark */
            font-size: 18px;
            font-weight: bold; /* Added font-weight to make the text bold */
            margin-bottom: 10px;
        }

        .button {
            display: inline-block;
            background-color: #007bff;
            color: #ffffff;
            font-size: 16px;
            font-weight: bold;
            padding: 12px 24px;
            text-decoration: none;
            border-radius: 4px;
            transition: background-color 0.3s ease;
        }

        .button:hover {
            background-color: #0056b3;
        }
    </style>
</head>
<body>
    <div class="container">
        <h1>Hello Everyone</h1>
        <p>This is a part of the WEEK6 #10WeeksOfCloudOps Challenge</p>
        <p>This is a sample web page deployed using Ansible Playbook</p>
        <a href="#" class="button">Get Started</a>
    </div>
</body>
</html>
```

Now we have to push the code into the git repository and then we need to integrate with Jenkins for running the Playbooks in a CI/CD model. This was the repository which is used for this project.

[Ansible-web-app](https://github.com/sridhar-modalavalasa/Ansible-web-app)

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1690542782499/9bed96ea-12f6-4295-827a-47e42daeb1a2.png align="center")

## Jenkins Pipeline:

This was the pipeline I created for the execution of the Ansible playbooks for automating the Ansible configuration management using Jenkins CI/CD.

```plaintext
pipeline {
    agent any 
    stages {
      stage('SCM Checkout') {
        steps {
           checkout([$class: 'GitSCM', branches: [[name: '*/main']], userRemoteConfigs: [[url: 'https://github.com/sridhar-modalavalasa/Ansible-web-app.git']]])  
        }
      }  
      stage('Execute Ansible') {
        steps {
           ansiblePlaybook credentialsId: 'ubuntu-slaves-key', disableHostKeyChecking: true, installation: 'ansible-copsc', inventory: 'hosts', playbook: 'web-app.yml'
        }  
      }
    }
}
```

The pipeline which is provided is a Jenkins pipeline, which is a collection of steps that can be used to automate the build, test, and deployment of software. In this case, the pipeline is used to build a web app from source code on GitHub, and then deploy the web app to a set of servers using Ansible.

### First Stage:

The first stage of the pipeline, SCM Checkout, checks out the source code from GitHub. The branches parameter specifies that the pipeline should only checkout the main branch of the repository. The userRemoteConfigs parameter specifies the URL of the repository.

### Second Stage:

The second stage of the pipeline, Execute Ansible, runs an Ansible playbook to deploy the web app. The credentialsId parameter specifies the ID of the credentials that Ansible will use to access the servers.

The disableHostKeyChecking parameter specifies that Ansible should not check the host keys of the servers. The installation parameter specifies the installation of Ansible to use.

The inventory parameter specifies the inventory file that Ansible will use. The playbook parameter specifies the playbook that Ansible will run.

The **ubuntu-slaves-key ID** is the details of the **pem key** which will be created when we are trying to access the servers. Try to paste the key in the below manner as shown in the image. Then we need to assign the **ID** and use it in the pipeline.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1690543516644/78428962-04d1-473d-a5f4-9abd17ccf8e7.png align="center")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1690543532329/3e23a58a-51ae-47db-8a71-159f056243c1.png align="center")

### For Generating the Pipeline Code:

Follow the below steps. After creating the pipeline name. Try to click on the **Pipeline syntax.** Then you will be able to see the image as shown below.

In the below image, I have chosen the ansible so that it will be able to generate a script for the playbook to run.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1690544374924/f5706558-5b45-4aff-8996-6895765c9d0d.png align="center")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1690544399665/5480ab8a-54a3-47bb-8df8-6020dd91238b.png align="center")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1690544419512/64ca4089-ec39-435d-855e-c8d41567420f.png align="center")

This will generate the script so that you will be able to access the code in the GitHub through Jenkins pipeline in the Ansible master server.

Click on **New Item** in the Jenkins server.

Assign the name for the pipeline and then proceed with the below steps as shown in the below image.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1690544910613/728bb8bf-b4e1-4a12-9bd6-a313cc653b58.png align="center")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1690544935219/4d04e6eb-9682-4622-a2c8-7c06587277cf.png align="center")

Click on the save and build the pipeline as shown in the below image. This was the Console Output which was shown below.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1690545067839/4b9683a5-4e48-4810-81f1-08d0a1f0bca8.png align="center")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1690545077174/3df12770-5281-4a58-a064-fd835cdecf2a.png align="center")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1690545087526/cb3e10fe-5da7-4c06-b95f-e9ed333614b7.png align="center")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1690545097608/848485b5-b1e7-406c-af02-3e1eb097ad04.png align="center")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1690545107495/f8a62f0a-2d40-4bf1-9bfb-f878495234a8.png align="center")

After the successful completion of the pipeline, you will be able to access the web application and the status of the Nginx server in the Linux servers.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1690545321617/cd3649c0-7a43-47d7-82f7-e8a25debd26b.png align="center")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1690545365121/e7371401-8283-430f-8dbd-de99de031edd.png align="center")

**Server-1:**

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1690545264485/39d3f3f1-f5d6-47dc-a90d-eb54fde4ba5a.png align="center")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1690545421908/adc34929-fa05-4e2f-a73e-f84122cb499d.png align="center")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1690545464795/00fa113e-6979-4df4-bc21-df877a9bd0ba.png align="center")

**Server-2:**

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1690545299097/67ed4e49-9ed1-43cb-aa34-3295229a5935.png align="center")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1690545440536/a53d7fd1-9276-486a-a17d-4ee5cacc76ef.png align="center")

## Conclusion:

Congratulations! You have successfully set up a Jenkins CI/CD pipeline that automates the installation of Nginx and deploys your application across multiple servers using Ansible. This streamlined process ensures consistent deployments and saves valuable time in managing server configurations. Happy automating!

### **Thank You**

I want to express my deepest gratitude to every one of you who has taken the time to read, engage, and support my journey.

Feel free to reach out to me if any corrections or add-ons are required on blogs. Your feedback is always welcome & appreciated.

~ SRIDHAR MODALAVALASA 😁🙌

[Follow me on Linkedin](https://www.linkedin.com/in/sridhar-modalavalasa-12b492173/)

[GitHub Repository](https://github.com/sridhar-modalavalasa)

## References:

[Ansible Reference](https://www.javatpoint.com/ansible)

[Ansible Detailed Overview](https://www.techbeatly.com/ansible-introduction/)