---
title: "Part1:Overview of CI/CD on Kubernetes Using Argo CD."
datePublished: Sun Jun 25 2023 05:47:39 GMT+0000 (Coordinated Universal Time)
cuid: cljb0dbsc0w1i4jnvaqd4cacf
slug: part1overview-of-cicd-on-kubernetes-using-argo-cd
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1687674436691/1036b37c-fa0a-4365-b34d-d13a340d9804.png
ogImage: https://cdn.hashnode.com/res/hashnode/image/upload/v1687674486452/2f42e6c3-bd78-4ce3-964b-b2537fc715ec.png
tags: cicd-on-kubernetes-using-argocd

---

### **GitOps:**

[GitOps](https://www.weave.works/technologies/gitops/) modernizes software management and operations by allowing developers to declaratively manage infrastructure and code using a single source of truth, usually a Git repository. Many development teams and organizations have adopted GitOps procedures to improve the creation and delivery of software applications.

It is a specific implementation of DevOps principles that focuses on using Git as the single source of truth for defining and managing the entire application lifecycle. It leverages the Git version control system to declaratively describe the desired state of the system and relies on automation to ensure that the actual state of the system matches the desired state.

### Kubernetes:

Kubernetes (commonly abbreviated as K8s) is an open-source container orchestration system for automating software deployment, scaling, and management1. It is a portable, extensible platform for managing containerized workloads and services that facilitates both declarative configuration and automation. It has a large and rapidly growing ecosystem with services, support, and tools widely available.

For a GitOps initiative to work, an orchestration system like [https://kubernetes.io/](https://kubernetes.io/) (**Kubernetes**) is crucial. The number of incompatible technologies needed to develop software makes Kubernetes a key tool for managing infrastructure. Without Kubernetes, implementing infrastructure-as-code (IaC) procedures is inefficient or even impossible. Fortunately, the wide adoption of Kubernetes has enabled the creation of tools for implementing GitOps.

### ArgoCD:

ArgoCD is an open-source continuous delivery tool specifically designed for Kubernetes. It helps in automating the deployment of applications and managing their lifecycle in a Kubernetes cluster. ArgoCD uses a declarative approach, where you define the desired state of your applications and let ArgoCD handle the deployment and synchronization with the actual state of the cluster.

One of these tools, [https://argoproj.github.io/cd/](https://argoproj.github.io/cd/) (**ArgoCD**) is a Kubernetes-native continuous deployment (CD) tool. It can deploy code changes directly to Kubernetes resources by pulling it from Git repositories instead of an external CD solution. Many of these solutions support only push-based deployments. Using ArgoCD gives developers the ability to control application updates and infrastructure setup from a unified platform. It handles the latter stages of the GitOps process, ensuring that new configurations are correctly deployed to a Kubernetes cluster.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1687667856078/1aed9010-2c84-4f85-8b73-10ac063a0bf6.png align="center")

We have an application that is running in Kubernetes that application is saying **Thank you for staying till end** which was the **Python-based flask application**. The magic starts happening when you change the code and push it to Git Hub.

As soon as you commit the changes a Jenkins job submitted automatically builds a new image pushes the new image to the docker hub and changes the deployment with the latest image id. The new image automatically gets deployed to the Kubernetes using GitOps and our application starts pointing to the new pod.

I have divided this project into **two parts** for your convenience which gives you a clear explanation. In both parts, I have covered these topics which are mentioned below.

### Topics Covered in the Whole Project:

* **GitOps workflow**
    
* **The Difference with the DevOps Workflow**
    
* **Dockerfile and Jenkinsfile walkthrough**
    
* **Jenkins installation and Jenkins job setup**
    
* **ArgoCD (GitOps) Installation**
    
* **ArgoCD (GitOps) Setup**
    
* **Automating Github to Jenkins using Webhook**
    

You are the developer who is working on code [app.py](http://app.py) and then you push this code to a repository in Github named. You have to create two repositories one is for **Python-Flask** and another is for **Kubernetes Manifest yaml** file. The below code is for the purpose for Python-flask purpose and name of the repository is **K8sArgoCD-flask-code.**

[Flask-code-repo](https://github.com/sridhar-modalavalasa/K8sArgoCD-flask-code)

As soon as you push the code the Jenkins job gets triggered which builds the docker container image and the name of the Jenkins job is **Building-image**. This job was created to create an image and push it to the docker hub repository.

It saves the image in a container registry in this we are using a docker hub and the name of the container image is **shreedhar4037/flask-app**. **The colon 5** is the tag for the container image and this part is important every time your code is changed to a new docker container image with the updated tag.

Coming back to GitHub we also have another repo for the Kubernetes manifest file the name of the repo is **K8s-Manifests-ArgoCD** and where we have deployment.yaml files. This deployment yaml files should reference this newly created container. I have given the reply link below.

[K8s-manifest-yaml-file](https://github.com/sridhar-modalavalasa/K8s-Manifests-ArgoCD)

How does that happen after Jenkins created this docker container image it will trigger the Jenkins job to update the manifest which will update the image in the deployment.yaml file. Now this deployment yaml file references **flask-app:5**

### GitOps Workflow:

We are using the GitOps  ArgoCD as the GitOps part tool but this approach will work as flux well.

GitOps continuously will monitor this Kubernetes manifest repo and if the state in the Kubernetes cluster deviates from the manifest files in the repo GitOps will grab those changes from the GitHub repository and deploy them into the Kubernetes cluster.

If there is no container running in the Kubernetes cluster and GitOps sees that so it is going to deploy the deployment.yaml file into the Kubernetes cluster.

So in the cluster, we have a container with the 5 running so this is for the first time. What if the application's application code [app.py](http://app.py) gets changed and pushed again? So in this case the job build image will build a new container image and save the new container image with the new tag flask:6.

Then this Jenkins job will trigger the update manifest Jenkins which will go and update the image reference in the deployment.yaml file. So now the Kubernetes cluster is running the container image with Flask 6 but the deployment yaml file is referencing the image with the tag 6. So Argocd will detect that and then it will terminate the container with the older tag and the container with the newest tag.

If you see in the above diagram these are the below things you can notice

**<mark>The Jenkins job that is building the docker container image and updating the manifest file is the CI or continuous integration. &nbsp;</mark>**  

**<mark>The GitOps that is deploying the deployment yaml file in the Kubernetes cluster is the CD or continuous deployment</mark>**<mark>. &nbsp;</mark>  

### The difference with the DevOps Workflow:

**How it will look like with the traditional DevOps without GitOps?**

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1687669806918/d3f73bdd-876f-4266-9c3b-754806e06d56.png align="center")

It is more of a push model so the job which is updating the manifest file can run kubectl commands to deploy those containers into the kubernetes clusters directly.

But what is the disadvantage of this well let's say someone deletes the deployment in the Kubernetes cluster it is not going to remediate automatically you need to somehow get alerted and then you need to rerun this update manifest Jenkins job to push changes back to the Kubernetes cluster.

Whereas GitOps as soon as something changes in the cluster. GitOps will automatically detect the cluster that is out of sync with the GitHub repository and it will automatically bring up the deployment other advantages will go through in this blog.

### **Summary GitOps:**

* Periodically syncs the running cluster with the desired state in Git Repo.
    
* GitOps will support both vanilla manifest files and Helm charts.
    
* Reduced learning cure than DevOps. Because it is utilizing the Kubernetes concepts under the hood gitOps is using Kubernetes operators.
    
* So you don’t need to learn different DevOps tools to implement the continuous deployment portion.
    
* Increased Security
    
* CI (Developer) and CD(Ops) permissions are separated with a Fewer number of components.
    

At last, GitOps doesn’t get rid of DevOps. This is the part I was referring to from traditional DevOps flow so if you have implemented the cd part using Jenkins.

you know you have to configure a lot of the stuff since GitHub is running within the Kubernetes cluster there are less number of components to create and manage and the last point is GitOps.

<mark>It does not mean getting rid of DevOps you still need to have the continuous integration part through DevOps also GitOps only comes into play when you have at least one cluster up and running to spin up that one cluster you have to use DevOps all right.</mark>

### GitOps vs DevOps:

### DevOps:

1. **Collaboration**: DevOps encourages close collaboration between development, operations, and other relevant teams, fostering a culture of shared responsibility and effective communication.
    
2. **Automation**: DevOps promotes the use of automation tools and practices to streamline the software development and deployment processes. This includes automating build, test, and deployment tasks to achieve faster and more reliable delivery.
    
3. **Continuous Integration and Continuous Delivery (CI/CD)**: DevOps emphasizes the integration of code changes frequently and continuously, coupled with automated testing and deployment. CI/CD pipelines automate the process of building, testing, and deploying applications, ensuring that changes can be quickly and reliably delivered to production environments.
    
4. **Infrastructure as Code (IaC):** DevOps encourages the use of infrastructure provisionings and management tools, such as configuration management systems and containerization technologies, to define and manage infrastructure and application environments as code.
    

### GitOps:

1. **Declarative Infrastructure and Application Management:** GitOps relies on declarative definitions stored in a Git repository to define the desired state of the infrastructure and applications. Any changes to the desired state are made by modifying the Git repository, and the GitOps tooling ensures that the changes are automatically applied to the system.
    
2. **Continuous Deployment and Synchronization:** GitOps tools continuously monitor the Git repository for changes and automatically deploy or synchronize the actual system state to match the desired state defined in the repository. This ensures that the system is always in the desired state and enables automated deployments without manual intervention.
    
3. **Pull-Based Model:** GitOps follows a pull-based model, where the system itself pulls the desired state from the Git repository and applies the necessary changes. This ensures that the system is self-contained and can recover from failures or inconsistencies by reconciling with the Git repository.
    
4. **Application Observability:** GitOps tools often provide visibility and observability features to monitor the health and status of deployed applications. They may integrate with monitoring and logging systems to provide real-time insights into the application's performance and health.
    

In summary, while DevOps is a broader philosophy and set of practices focused on collaboration, automation, and continuous delivery, GitOps is a specific implementation of DevOps that emphasizes using Git as the single source of truth and relies on a declarative approach to manage the application lifecycle. GitOps can be seen as a specialized subset of DevOps that leverages Git's strengths and promotes a self-contained, automated, and auditable system for application deployment and management.

### References:

[https://argo-cd.readthedocs.io/en/stable/](https://argo-cd.readthedocs.io/en/stable/)

[https://kubernetes.io/](https://kubernetes.io/)

[https://www.youtube.com/watch?v=eqiqQN1CCmM](https://www.youtube.com/watch?v=eqiqQN1CCmM)