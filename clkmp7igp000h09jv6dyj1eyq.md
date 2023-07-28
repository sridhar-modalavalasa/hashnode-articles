---
title: "Automating MongoDB Installation Across Multiple Servers using Ansible Roles & Jenkins."
datePublished: Fri Jul 28 2023 14:48:09 GMT+0000 (Coordinated Universal Time)
cuid: clkmp7igp000h09jv6dyj1eyq
slug: automating-mongodb-installation-across-multiple-servers-using-ansible-roles-jenkins
tags: ansible-roles-for-mongodb-using-jenkins-pipeline

---

## Architecture:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1690550711841/e011ab05-d3f8-4e97-b4eb-b11c507d3da4.png align="center")

## Ansible Roles:

In Ansible, roles are a powerful way to organize and encapsulate the playbook logic, making it easier to manage complex infrastructure configurations and promote code reusability. A role is essentially a collection of tasks, variables, templates, and files that can be applied to a specific set of hosts in a playbook. It allows you to modularize your configurations, making them more maintainable and scalable.

### **Role directory structure:**

Inside the roles directory, you typically create subdirectories, each representing a separate role. The role directories usually follow a specific structure, including the following directories:

**defaults:** This directory contains default variables for the role. These variables are used if no other value is specified. files: Here, you can place files that need to be transferred to the managed hosts during playbook execution.

**handlers:** Handlers are tasks that are triggered by other tasks. This directory holds the handler definitions.

**meta:** This directory can contain metadata about the role, such as dependencies on other roles.

**tasks:** The main tasks for the role are defined in this directory. These tasks are executed on the managed hosts.

**templates:** If you have files that need to be dynamically generated based on variables, you can place their templates in this directory.

**vars:** Variables specific to the role can be defined in this directory.

### **Advantages of Using Ansible Roles:**

**Modularity**: Roles promote a modular approach, enabling you to break down your playbook into smaller, manageable units. This makes it easier to understand and maintain your infrastructure configurations.

**Modularity:** Roles promote a modular approach, enabling you to break down your playbook into smaller, manageable units. This makes it easier to understand and maintain your infrastructure configurations.

**Reusability:** Roles can be reused across multiple playbooks and projects, reducing duplication of code and ensuring consistency in configurations.

**Abstraction:** Roles abstract the complexity of your configurations, making your playbooks cleaner and more focused on the desired end-state.

**Team Collaboration:** Roles facilitate collaboration among team members, as each role can be developed independently and then integrated into the main playbook.

The configuration settings and the remaining things are the same as mentioned in the below blog.

[Configuration settings](https://shreedhar1998.hashnode.dev/automating-nginx-installation-and-application-deployment-across-multiple-servers-using-ansible-jenkins)

You will find out the above link details related to the SSH Connectivity.

Now we will jump into the roles which I have created for the MongoDB installation using the Jenkins pipeline and Ansible.

Now we will look into the roles playbook and the directory structure as shown below. For creating the roles, we have to use the below command.

```plaintext
$ ansible-galaxy init <role-name>
$ ansible-galaxy init mongodb
$ ansible-galaxy init mongodb_users
```

I have created one directory in that by using the above command I have created roles. Now we will go and check the two modules for creating the MongoDB in Ubuntu servers.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1690551438057/247f0b8b-e695-4608-9c65-6876e10c40c8.png align="center")

[GitHub Repository](https://github.com/sridhar-modalavalasa/Ansible-Mongodb-roles)

## MongoDB Role:

### Mongodb role --&gt; tasks --&gt; main.yaml

```plaintext
---
- name: Update apt cache
  apt:
    update_cache: yes

- name: Check Ubuntu version
  command: lsb_release -rs
  register: ubuntu_version

- name: Check if focal-security repository exists
  stat:
    path: /etc/apt/sources.list.d/focal-security.list
  register: focal_security_repo

- name: Add focal-security repository
  shell: echo "deb http://security.ubuntu.com/ubuntu focal-security main" | sudo tee /etc/apt/sources.list.d/focal-security.list
  when: ubuntu_version.stdout == "22.04" and not focal_security_repo.stat.exists

- name: Update apt cache
  apt:
    update_cache: yes
  when: ubuntu_version.stdout == "22.04"

- name: Install libssl1.1
  apt:
    name: libssl1.1
    state: present
  when: ubuntu_version.stdout == "22.04"

- name: Installing dependencies
  package:
    name: "{{ item }}"
    state: present
    update_cache: yes
  loop:
    - curl
    - gnupg
    - python3-pip
  become: yes

- name: Install pymongo
  pip:
    name: pymongo

- name: Check if MongoDB APT keyring exists
  stat:
    path: /usr/share/keyrings/mongo-key.gpg
  register: mongo_keyring_exists

- name: Add MongoDB APT keyring
  shell: "curl -fsSL https://www.mongodb.org/static/pgp/server-{{ mongodb_version }}.asc | sudo gpg --dearmour -o /usr/share/keyrings/mongo-key.gpg"
  args:
    executable: /bin/bash
  when: not mongo_keyring_exists.stat.exists

- name: Check if MongoDB repository exists
  stat:
    path: /etc/apt/sources.list.d/mongodb-org-{{ mongodb_version }}.list
  register: mongo_repository_exists

- name: Add MongoDB repository
  shell: sudo sh -c 'echo deb [ arch=amd64,arm64 signed-by=/usr/share/keyrings/mongo-key.gpg] https://repo.mongodb.org/apt/ubuntu focal/mongodb-org/{{ mongodb_version }} multiverse > /etc/apt/sources.list.d/mongodb-org-{{ mongodb_version }}.list'
  args:
    executable: /bin/bash
  when: not mongo_repository_exists.stat.exists

- name: Update apt cache
  apt:
    update_cache: yes

- name: Install MongoDB packages
  apt:
    name: mongodb-org
    state: latest
- name: Enable and start MongoDB service
  service:
    name: mongod
    state: started
    enabled: yes

- name: Enable remote connections in MongoDB
  lineinfile:
    path: /etc/mongod.conf
    regexp: '^ *bindIp:.*'
    line: '  bindIp: 0.0.0.0'
    state: present
    backup: yes
  notify:
    - Restart MongoDB Service

- name: Enable authentication in MongoDB
  lineinfile:
    path: /etc/mongod.conf
    line: "security:\n  authorization: enabled"
    insertafter: "^#security:"
    state: present
    backup: yes
  notify: 
    - Restart MongoDB Service
```

### **Explanation:**

* The playbook is a set of Ansible tasks that will install and configure MongoDB on an Ubuntu 22.04 server. The playbook first checks the Ubuntu version and, if it is 22.04, adds the focal-security repository. This repository contains security updates for Ubuntu 22.04.
    
* The playbook then installs the following dependencies which are curl, gnupg, python3-pip. These dependencies are needed to install pymongo, the Python client for MongoDB.
    
* The playbook then checks if the MongoDB APT keyring exists. If it does not exist, the playbook downloads the keyring and imports it.
    
* The playbook then checks if the MongoDB repository exists. If it does not exist, the playbook creates the repository file.
    
* The playbook then updates the apt cache and installs the MongoDB packages.
    
* The playbook then enables and starts the MongoDB service.
    
* The playbook then enables remote connections in MongoDB by setting the bindIp option to 0.0.0.0 in the /etc/mongod.conf file.
    
* The playbook then enables authentication in MongoDB by adding the following line to the /etc/mongod.conf file.
    
* Then it will restart the service of the MongoDB which is defined in the handlers.
    

### Mongodb role --&gt; handlers --&gt; main.yaml

```plaintext
---
- name: Restart MongoDB Service
  service:
    name: mongod
    state: restarted
```

The above code is just for restarting the service.

### Mongodb role --&gt; defaults --&gt; main.yaml

```plaintext
---
mongodb_version: "6.0"
```

In this particular code, we have defined a mongodb\_version for installation in the Ubuntu 22.04 LTS server.

## Mongodb\_users role:

### Mongodb\_users role --&gt; tasks --&gt; main.yaml

```plaintext
---
- name: Create MongoDB root user
  mongodb_user:
    login_port: "27017"
    database: "admin"
    name: "{{ mongodb_root_user }}"
    password: "{{ mongodb_root_password }}"
    roles: "root"

- name: Create MongoDB administrative user siteUserAdmin
  mongodb_user:
    login_user: "{{ mongodb_root_user }}"
    login_password: "{{ mongodb_root_password }}"
    login_port: "27017"
    database: "{{ database_name }}"
    name: "{{ mongodb_admin_user }}"
    password: "{{ mongodb_admin_password }}"
    roles:
      - { db: "admin", role: "readWrite" }
      - { db: "{{ database_name }}", role: "readWrite" }

- name: Create MongoDB backup user siteUserBackup
  mongodb_user:
    login_user: "{{ mongodb_root_user }}"
    login_password: "{{ mongodb_root_password }}"
    login_port: "27017"
    database: "{{ database_name }}"
    name: "{{ mongodb_backup_user }}"
    password: "{{ mongodb_backup_password }}"
    roles:
      - { db: "admin", role: "backup" }
```

## Explanation:

### Task 1: Creates a MongoDB root user

* The root user has full access to the MongoDB database, including the ability to create and modify other users. The login\_port option specifies the port that the MongoDB server is listening on.
    
* The database option specifies the database where the user will be created. The name option specifies the name of the user. The password option specifies the password for the user.
    
* The roles option specifies the roles that the user will be assigned. In this case, the root user is assigned the root role, which gives the user full access to the MongoDB database.
    

### Task 2: Creates a MongoDB administrative user

* The administrative user has read-write access to the MongoDB database. The login\_user option specifies the name of the user who will be used to authenticate to the MongoDB server.
    
* The login\_password option specifies the password for the user who will be used to authenticate to the MongoDB server. The login\_port option specifies the port that the MongoDB server is listening on.
    
* The database option specifies the database where the user will be created. The name option specifies the name of the user. The password option specifies the password for the user.
    
* The roles option specifies the roles that the user will be assigned. In this case, the administrative user is assigned the read-write role for both the admin database and the database\_name database.
    

### Task 3: Creates a MongoDB backup user

* The backup user has read-only access to the MongoDB database. The login\_user option specifies the name of the user who will be used to authenticate to the MongoDB server.
    
* The login\_password option specifies the password for the user who will be used to authenticate to the MongoDB server. The login\_port option specifies the port that the MongoDB server is listening on.
    
* The database option specifies the database where the user will be created. The name option specifies the name of the user. The password option specifies the password for the user.
    
* The roles option specifies the roles that the user will be assigned. In this case, the backup user is assigned the backup role for the admin database.
    

### Mongodb\_users role --&gt; vars --&gt; main.yaml

```plaintext
---
database_name: "test_db"

mongodb_root_user: "root"
mongodb_root_password: "root1234"

mongodb_admin_user: "superadmin"
mongodb_admin_password: "superadmin123"

mongodb_backup_user: "backupuser"
mongodb_backup_password: "backupuser123"
```

Here in the above yaml. We are just defining the variables which are mentioned in the **tasks --&gt; main.yaml**

These were the roles that I have created. Now we have to define the hosts and call the roles from one yaml file which is outside of these directories.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1690551438057/247f0b8b-e695-4608-9c65-6876e10c40c8.png align="left")

### hosts file:

```plaintext
[my-servers]
server1 ansible_host=172.31.80.149
server2 ansible_host=172.31.83.39

[all:vars]
ansible_python_interpreter=/usr/bin/python3
ansible_user=ubuntu
```

### Mongo-db.yaml:

```plaintext
- name: Install and configure MongoDB on Ubuntu 22.04 
  hosts: my-servers
  become_user: root
  become_method: sudo
  become: true
  roles:
    - mongodb
    - mongodb_users
```

With this, we have created the roles and playbook for calling the role outside of the role. Now we have to automate this thing through the Jenkins pipeline.

## Jenkins Pipeline

The same pipeline I created for the previous project which I worked on the below blog:

You will find out these section configuration details in the Jenkins setup.

[Jenkins Pipeline configuration details and explanation](https://shreedhar1998.hashnode.dev/automating-nginx-installation-and-application-deployment-across-multiple-servers-using-ansible-jenkins)

```plaintext
pipeline {
    agent any 
    stages {
      stage('SCM Checkout') {
        steps {
           checkout([$class: 'GitSCM', branches: [[name: '*/main']], userRemoteConfigs: [[url: 'https://github.com/sridhar-modalavalasa/Ansible-Mongodb-roles.git']]])  
        }
      }  
      stage('Execute Ansible') {
        steps {
           ansiblePlaybook credentialsId: 'ubuntu-slaves-key', disableHostKeyChecking: true, installation: 'ansible-copsc', inventory: 'hosts', playbook: 'mongo-db.yaml'
        }  
      }
    }
}
```

Now Click on the New **Item** in the Jenkins UI. Then create a name and then configure it like this as shown below.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1690553940056/67b3227f-46e5-4bed-8272-efe761444b54.png align="center")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1690553972474/e2b827b0-e34e-4997-b980-63af995cd3a3.png align="center")

Then Click on the **save** and then click on the **Build** option.

This was the console output for the above pipeline. I have created for the ansible roles to execute through Jenkins Pipeline.

### **Console Output:**

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1690554164699/8f43b8a8-ca57-4bf9-a35a-b857d63dd5a4.png align="center")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1690554174040/1bfbde47-30d0-4bdd-a415-bd214602ca1a.png align="center")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1690554183638/d0bb7976-7816-4238-94ec-6733525f815c.png align="center")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1690554193235/1ca805e9-3cd7-4183-8f8c-dd4b24750f2c.png align="center")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1690554201841/5d9cd2f3-00dd-4336-87ac-174e93601cf4.png align="center")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1690554210260/687c7609-15f4-4520-80a5-862bb15436d5.png align="center")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1690554217661/d1314306-e2e0-4100-9b2b-07f73f03943d.png align="center")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1690554224903/ac23e260-8684-4254-8586-933ef6a22c27.png align="center")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1690554246726/d36c44c3-2ae8-4feb-a2da-ee6baf6592a4.png align="center")

### Server-1:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1690554295694/3b6cb3a8-f3e6-40ab-88fc-c901f1685cc9.png align="center")

If you have observed the above image. The **MongoDB** has been installed and the users also created without the issues. Now we can use this for the database purpose.

### Server-2:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1690554409512/448af53f-3faa-4738-a4c9-ebb90beae1c7.png align="center")

## **Conclusion:**

Congratulations! You have successfully set up a Jenkins CI/CD pipeline that automates the installation of MongoDB application across multiple servers using Ansible. This streamlined process ensures consistent deployments and saves valuable time in managing server configurations. Happy automating!

## **Thank You**

I want to express my deepest gratitude to every one of you who has taken the time to read, engage, and support my journey.

Feel free to reach out to me if any corrections or add-ons are required on blogs. Your feedback is always welcome & appreciated.

~ SRIDHAR MODALAVALASA 😁🙌

[Follow me on Linkedin](https://www.linkedin.com/in/sridhar-modalavalasa-12b492173/)

[GitHub Link](https://github.com/sridhar-modalavalasa)

## **References:**

[Ansible Overview and Lessons](https://www.techbeatly.com/ansible-introduction/)