---
title: "PART 1: Simplified Dependency for EKS Cluster, Observability stack, and Dockerizing the Nodejs Application."
datePublished: Sun Jul 16 2023 08:04:10 GMT+0000 (Coordinated Universal Time)
cuid: clk55a2tp0g6nfgnvfpwhce51
slug: part-1-simplified-dependency-for-eks-cluster-observability-stack-and-dockerizing-the-nodejs-application
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1689484854373/11532142-cc94-4895-a482-a8652740373d.png
tags: dependancies-of-prometheus-and-grafana-in-eks

---

## What is Prometheus?

[Prometheus](https://prometheus.io/) is an open-source automated monitoring and alerting system. It has become a widely accepted tool for monitoring highly dynamic container environments such as Kubernetes and Docker Swarm.

[Kubernetes](https://kubernetes.io/)

[Docker Swarm](https://docs.docker.com/engine/swarm/)

It can collect metrics from various sources, including containers, servers, and applications, and store them in a time-series database. Prometheus provides a flexible query language, called PromQL, that allows you to retrieve and analyze data. It also includes a web interface and an API for data interaction.

## What is Grafana?

[Grafana](https://grafana.com/) is a multi-platform that gets data from a data source such as Prometheus and transforms it into visualizations charts. We can create our dashboards or use the existing ones provided by Grafana. We can personalize the dashboards as per our requirements.

## What is Helm?

[Helm](https://helm.sh/) is the package manager for Kubernetes. Helm Charts **help you define, install, and upgrade even the most complex Kubernetes application**. Charts are easy to create, version, share, and publish — so start using Helm and stop the copy-and-paste.

This article will teach you how to integrate Prometheus and Grafana on Kubernetes using Helm.

## Prerequisites:

* AWS account with the access key and secret key.
    
* EC2 Instance with AWS CLI, EKSCTL, Kubectl, and Helm chart.
    
* Kubernetes Cluster, I am using EKS Cluster for the Demo Purpose.
    
* Dockerized Nodejs Application in Docker hub.
    
* Prometheus and Grafana Setup.
    

## **Setup an AWS EC2 Instance:**

Log in to an AWS account using a user with admin privileges and ensure your region is set to us-east-1 N. Virginia.

Move to the EC2 console. Click Launch Instance.

For name use My-Eks-Cli.

Select AMIs as Ubuntu and select Instance Type as t2.micro. Create new Key Pair and Create a new Security Group with traffic allowed from SSH, HTTP, and HTTPS.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1689486045099/3eabd9f8-e1d4-4ae8-bdf5-a5fe0b2307ee.png align="center")

Click on Launch Instance and once EC2 Instance started, connect to it with EC2 Instance Connect.

## **Install AWS CLI and Configure:**

Now we need to set up the AWS CLI on the EC2 machine so that we can use ***eksctl*** in the later stages

Let us get the installation done for ***AWS CLI 2.***

**Linux x86(64-bit)** If you are using ***Linux x86(64-bit)*** operating system:

```plaintext
$ curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip" 
$ sudo apt install unzip
$ unzip awscliv2.zip 
$ sudo ./aws/install
```

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1689486270797/013b7346-9b47-4abc-9ce4-38d9ccb9f3a6.png align="center")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1689486285065/7f47b513-1776-4e04-957c-03360c3c158c.png align="center")

Okay now after installing the AWS CLI, let's configure the ***AWS CLI*** so that it can authenticate and communicate with the AWS environment.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1689486613875/bf672b82-b9f3-4ef8-9d25-6c332b7f2c30.png align="center")

## **Install and Setup Kubectl:**

Moving forward now we need to set up the [**kubectl**](https://kubernetes.io/docs/reference/kubectl/overview/) also onto the EC2 instance.

```plaintext
$ curl -O https://s3.us-west-2.amazonaws.com/amazon-eks/1.27.1/2023-04-19/bin/linux/amd64/kubectl
$ chmod +x ./kubectl
$ mkdir -p $HOME/bin && cp ./kubectl $HOME/bin/kubectl && export PATH=$HOME/bin:$PATH
$ echo 'export PATH=$HOME/bin:$PATH' >> ~/.bashrc
$ kubectl version --short --client
```

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1689486717544/7f778bba-a3ba-4d48-a761-8a015c2595b3.png align="center")

## **Install and Setup eksctl:**

Download and extract the latest release of eksctl following command.

```plaintext
$ ARCH=amd64
$ PLATFORM=$(uname -s)_$ARCH
$ curl -sLO "https://github.com/weaveworks/eksctl/releases/latest/download/eksctl_$PLATFORM.tar.gz"
$ tar -xzf eksctl_$PLATFORM.tar.gz -C /tmp && rm eksctl_$PLATFORM.tar.gz
$ sudo mv /tmp/eksctl /usr/local/bin

# Check the eksctl version.
$ eksctl version
```

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1689486844658/da1a7269-5ebc-46f8-a7ae-f9c8c7f8b56b.png align="center")

## **Install Helm chart:**

The next tool we need is Helm Chart. Helm is a package manager for Kubernetes, an open-source container orchestration platform. Helm helps you manage Kubernetes applications by making it easy to install, update, and delete them.

**Install Helm Chart** - Use the following script to install the helm chart -

```plaintext
$ curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3
$ chmod 700 get_helm.sh
$ ./get_helm.sh
```

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1689487039893/e716b3ad-9840-43d4-8112-2f18dd55e3df.png align="center")

This way we install all AWS CLI, kubectl, eksctl, and Helm. Now we need to set up the Kubernetes Cluster setup. I am using the EKS Cluster service in AWS.

# **Creating an Amazon EKS cluster using eksctl:**

Now in this step, we are going to Create [Amazon EKS clusters](https://docs.aws.amazon.com/eks/latest/userguide/clusters.html) using eksctl

You need the following to run the eksctl command:

1. **Name of the cluster :** my-eks-monitoring
    
2. **Version of Kubernetes :** --version 1.24
    
3. **Region :** --region us-east-1
    
4. **Nodegroup name/worker nodes :** --nodegroup-name worker-nodes
    
5. **Node Type :** --nodegroup-type t3.medium
    
6. **Number of nodes:** --nodes 2
    
7. **Minimum Number of nodes:** --nodes-min 2
    
8. **Maximum Number of nodes:** --nodes-max 3
    

Here is the eksctl command -

```plaintext
eksctl create cluster --name my-eks-monitoring --version 1.24 --region us-east-1 --nodegroup-name worker-nodes --node-type t3.medium --nodes 2 --nodes-min 2 --nodes-max 3
```

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1689488101459/99ca2d8d-67c2-4b6b-9b6c-61f43dcd9023.png align="center")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1689488260210/f6e2d185-90fb-4edf-9e03-1e55fc9ae9ad.png align="center")

It took me 20 minutes to complete this EKS cluster. If you get any error for not having sufficient data for mentioned availability zone then try it again.

```plaintext
aws eks update-kubeconfig --name my-eks-monitoring
```

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1689489196401/ad22dc7b-135c-46b3-8671-79fbbab868e8.png align="center")

Verify the EKS Kubernetes cluster on AWS Console.

You can go back to your AWS dashboard and look for **Elastic Kubernetes Service -&gt; Clusters**

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1689488377480/2922912c-cd0f-482c-8455-9ba938e23104.png align="center")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1689488441596/6c5b5642-4170-42fc-8502-aeeee608c1ee.png align="center")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1689488503425/faae13e6-e80f-4c6b-b738-3ff48159f94c.png align="center")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1689488545238/00552e11-f3e5-487b-94ac-2c64cc4d9017.png align="center")

Now we had to check in the same server in which we have installed all the utilities which are required for this project.

In the same server run these commands to check whether the cluster is running fine or not.

```plaintext
$ kubectl get all --all-namespaces
```

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1689488818731/d3c77da6-d956-4603-a4cf-ae4a88bbf0c7.png align="center")

# **Installing the Kubernetes Metrics Server:**

Alright, the next step would be to install the Kubernetes Metrics server onto the Kubernetes cluster so that Prometheus can collect the performance metrics of Kubernetes.

Deploy the Metrics Server with the following command:

```plaintext
kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml
```

```plaintext
 $ kubectl get deployment metrics-server -n kube-system
```

Verify that the metrics-server deployment is running the desired number of pods with the following command.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1689489391043/6cf626d8-67ca-47c7-a667-88e0b1080967.png align="center")

## Dockerizing Nodejs Application:

I am pasting the link below for the application code. I am using this Nodejs Application for this project.

[GitHub Nodejs Application](https://github.com/sridhar-modalavalasa/Nodejs-circleci)

Then we need to create a Dokcerfile for the above application which we want to use in the Prometheus and Grafana for metrics scraping.

```plaintext
# Set the base image to use for subsequent instructions
FROM node:alpine
# Set the working directory for any subsequent ADD, COPY, CMD, ENTRYPOINT,
# or RUN instructions that follow it in the Dockerfile
WORKDIR /usr/src/app
# Copy files or folders from source to the dest path in the image's filesystem.
COPY package*.json /usr/src/app/
COPY . /usr/src/app/
# Execute any commands on top of the current image as a new layer and commit the results.
RUN npm install --production
# Define the network ports that this container will listen to at runtime.
EXPOSE 3000
# Configure the container to be run as an executable.
ENTRYPOINT ["npm", "start"]
```

Now I will run this application in my docker environment for checking whether this application is running or not.

```plaintext
docker build .
```

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1689490074904/718c9b25-d142-44c8-9071-a1387e4e7f5f.png align="center")

Now we will check with the below command.

```plaintext
docker images
```

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1689490173622/f0a53ef0-bafc-44f5-93dd-7e82ff6e0343.png align="center")

You can verify that the image is created. Now we have to create a tag for this image and Repository name by using the below command.

```plaintext
docker tag <existing-image-id> <new-repository-name>:<new-tag-name>
```

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1689490336393/85c46bf4-8a64-4d6d-a393-cb974bb52503.png align="center")

Now we have to run this image as a container and we need to check whether the application is running fine or not on the web.

This was the command I used to run this image as a container as shown in the below command.

```plaintext
docker run -it -p 3000:3000 eks-nodejapp:latest
```

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1689490684386/05a81334-15a5-467f-8f3a-2f651b5d0eef.png align="center")

Now we will check on the web with the localhost means here is the server IP Address we will use in the web as shown in the below image.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1689490577887/7dfc67a5-3990-4a8f-81ac-08b9780bf94f.png align="center")

Now we have dockerized our application in the Docker. Now we will push this image into the Dockerhub by using the below commands shown below.

First of all, create a Dockerhub account with username and password and then create a Dockerhub repository as shown in the below image.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1689491051428/31c9b1bf-0d48-4701-adfd-f5fccb1974ae.png align="center")

With the help of the above repository, we will push our docker image into the Docker hub account and then we will use that image in the Kubernetes Manifest file for running our application in the K8s cluster.

```plaintext
$ docker login 
###username and password will ask
$ docker tag <image-name> <dockerhub-username>/<repository-name>:<tag-name>
$ docker push <dockerhub-username>/<repository-name>:<tag-name>
```

By using these commands we will push our image into the Dockerhub. Then we need to setup the Prometheus and Grafana in EKS Cluster to set up the Observability for the NodeJS Application.

We will see further setup of Prometheus and Grafana in the EKS Cluster using Helm charts. Scraping the metrics using Prometheus and visualizing the dashboards in Grafana.

### References:

[https://prometheus.io/](https://prometheus.io/)

[https://grafana.com/](https://grafana.com/)

[https://kubernetes.io/](https://kubernetes.io/)

Follow this blog channel for further interesting topics in the

### #10weeksofcloudops

[Follow me on Linkedin](https://www.linkedin.com/in/sridhar-modalavalasa-12b492173/)

[Follow me on GitHub](https://github.com/sridhar-modalavalasa)