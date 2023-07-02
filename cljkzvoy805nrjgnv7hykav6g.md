---
title: "Automation of NodeJS Deployment using Ansible playbook."
datePublished: Sun Jul 02 2023 05:31:39 GMT+0000 (Coordinated Universal Time)
cuid: cljkzvoy805nrjgnv7hykav6g
slug: automation-of-nodejs-deployment-using-ansible-playbook
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1688270576681/63ee5c17-dab5-4922-94ce-41f1a5f5ee9d.jpeg
tags: deployment-of-nodejs-using-ansible

---

### **What is Ansible :**

* **Ansible is an open-source software provisioning, configuration management, and application-deployment tool enabling infrastructure as code.** It runs on many Unix-like systems and can configure both Unix-like systems as well as Microsoft Windows.
    
* Ansible was written by Michael DeHaan and **acquired by Red Hat in 2015.** It includes declarative language to describe system configuration. **Ansible is agentless**, temporarily connecting remotely via SSH or Windows Remote Management to do its tasks.
    

### **Basic Architecture of Distributed Ansible Cluster :**

* Before installing software we need to understand **why and how we use Ansible.**  
    Normally **in industry,** we have **thousands of servers** and **manually configuring those servers** is a very **haptic and time-consuming task**.  
    Here the play of Ansible comes. We can **configure any kind of server using Ansible in just a few minutes** and also We can configure thousands of servers in parallel using Ansible.
    

**Let's understand the below-shown architecture of Ansible.**

![](https://miro.medium.com/v2/resize:fit:700/0*f-YX6KxLIOGMy3NQ.png align="left")

* As you can notice We have an **“Ansible Management Node”** and we have some **“Host”** in the diagram. **You can think this Ansible Management Node is our Master Node and the Host node is the worker node.** That means in the worker node, we are configuring our servers.
    
* I already told you that **Ansible can configure any kind of server and is also capable of doing it to thousands of servers in parallel**. So, in the diagram, we can see **Ansible is using SSH protocol to reach every server at the same time.**
    
* In Ansible rather than using Master, we use the term **“Controller Node”** and for the worker, we use the term **“Managed Node”**. Also after installing Ansible, I will talk about “inventory” and “playbook”, so don't worry about those things.
    

## **Installing Ansible software on RHEL8:**

We just need to run only one single command. But before running that command one simple note.

**<mark>Ansible is written using Python language and by default in RHEL8, python 3 is installed, but in your system, if it's not present then install python at first. Next, run the below-mentioned command and Ansible will be installed on your RHEL8 system.</mark>**

```plaintext
pip3 install ansible

(or) you can use the below commands as well. 

yum update -y 
yum install ansible 
```

### Setting up Hosts/Inventory :

Next, we going to set up the hosts or managed nodes' details in our Master Node. **In Ansible rather than using hosts, we use the term “Inventory”.**

Remember one thing that in Ansible, whatever we do, we always use Controller Node. **We never go to any Managed Node.** But Ansible will configure our managed nodes to work as servers. So, **Ansible needs the IP of those managed nodes.**

That's why we need to create one file inside our controller system where we gonna mention the IP address of those managed nodes. **To make things simple here I am using one managed node.** But once you learn more, you will be able to work with multiple managed nodes and will be able to group them.

**Run these below-mentioned commands and it will create your host's file.**

```plaintext
mkdir Ansible
cat > home/User/Ansible/hosts
<your server Ip address> ansible_user=<your user> ansible_password=<your password>
```

* Here at first, we created one folder called **“/etc/ansible” because by default Ansible uses this folder for storing configuration files,** which I will show soon.
    
* Next, as you might have previously noticed that Ansible uses SSH protocol to execute anything on Managed Node so **in the “hosts” file I mentioned the IP of the managed node and then the username (root) and password (redhat) of my managed node.** To make things simple here I directly used the “ansible\_user” and “ansible\_password” keywords.
    
* Here I used the “cat” command to create the file. You can use VI or gedit editor also.
    
* **To know more about Ansible Inventory :**
    

[Inventory](https://docs.ansible.com/ansible/latest/user_guide/intro_inventory.html)

Now you have to make a connection between the two servers. Then you have to generate a key as a user in Linux.

<mark>Generate the keys in the master and Worker nodes in User only not in root user.</mark>

```plaintext
ssh key-gen
```

After generating the key in the Master node and worker node follow the below steps mentioned below.

**Master Node: Try to generate a key in User not in Root user.**

```plaintext
cat ~/.ssh/id_rsa.pub
```

**Try to copy the output and paste it to the worker node as mentioned in the below steps.**

**Worker Node: Try to follow the steps in User not in root user.**

```plaintext
cd ~/.ssh/ 
vi authorized_keys
```

**The key which you copied in the master node should have been paste in this location. If the file is not created in a certain specific directory. Try to create a file and paste it into the specific location as mentioned above.**

<mark>After following the above steps. You have to be connected to the target machine without a username and password.</mark>

```plaintext
ssh <your target machine IP>
```

Now I am trying to explain the playbook for the automation of Nodejs Deployment which I have used for this project.

```plaintext
- name: Install node and npm
  hosts: <your remote Ip>
  become: true
  become_user: root
  tasks:
  - name: Update yum repo and cache
    yum: update_cache=yes
  - name: Install nodejs and npm
    yum:
      name:
      - nodejs
      - npm

- name: Deploy nodejs app
  hosts: <your remote ip>
  vars_files:
    - project-vars 
  tasks:
  - name: Unpack the nodejs file
    unarchive:
      src: "{{location}}"
      dest: "{{destination}}"
      remote_src: yes
  - name: Install Dependencies
    npm:
      path: "{{destination}}/{{directory}}/"
  - name: Start the application
    command:
      chdir: "{{destination}}/{{directory}}/"
      cmd: node app
    async: 1000
    poll: 0
  - name: Ensure that app is running
    shell: ps aux | grep node
    register: app_status
  - debug:
      msg: "{{app_status.stdout_lines}}"
```

### **In the above playbook, we define two plays.**

**Play 1: Install node and npm**

This play installs the Node.js runtime and the Node Package Manager (npm) on the remote host. It uses the **yum** module to update the apt repository and cache, and then install the **node js and npm** packages.

For installing the packages I have used the **become\_user=root argument**

**Play 2: Deploy the nodejs app**

This play deploys the Node.js application on the remote host. I have defined hosts.

**var\_files:** In this particular module, the file I have defined the variables which are required for doing the tasks which are mentioned in the play and which will stay outside of the playbook and I am calling here.

**tasks:** In this section, we will define our tasks as mentioned above.

* Unpack the nodejs file using the unarchive module.
    
* **src:** In this module, it is the location where my Tar file of the Nodejs application resides. You have to mention your tar file path here. I am pasting my GitHub link repo for the Nodejs application purpose.
    

[NodeJS application GitHub Repository](https://github.com/sridhar-modalavalasa/Nodejs-CircleCI)

I will share the steps for making this whole application as a Tar file. You can follow the below steps for making.

```plaintext
tar cvf  Nodejs-app.tar  Nodejs-circleci/
gzip Nodejs-app.tar
```

* **dest:** In this module**,** it is the location of the remote server where you want to untar the file and install your application dependencies and you want to run the application in the remote server.
    
* **npm:** I this module, Installing the dependencies using this in the remote server.
    
* **chdir:** Then I am changing my directory location using this module, in which my application is untared and installed the dependencies in the remote server.
    
* **cmd:** By using this module I am trying to start my application on a remote server.
    
* In the last task, I am checking whether my application is append running or not in the remote server using some process checking commands.
    
* I am storing the task details in the msg module as you can see in the playbook.
    

Now we have to run this playbook using the below command.

```plaintext
ansible-playbook -i hosts My-Nodejs-app.yml 
```

This will initialize your application and you have to check the web using the port number 1337. Because that was the application port I mentioned in my Nodejs Code. I have used CircleCI Image for getting a good view.

you have to check with the **<mark>http://&lt;your remote server Ip&gt;:1337</mark>**

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1688274881120/73f6a981-39a0-4aee-b400-dc687f70e6a8.png align="center")

Hope you guys have enjoyed it, Thanks for staying still until the end.

### References:

[GitHub repository of this project application](https://github.com/sridhar-modalavalasa/Nodejs-CircleCI)

[Ansible Tutorials](https://www.youtube.com/watch?v=3RiVKs8GHYQ&list=PLT98CRl2KxKEUHie1m24-wkyHpEsa4Y70)