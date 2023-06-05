---
title: "Stress Command Utilization in Linux and targeting Kubernetes pods"
seoTitle: "Stress command in both Linux and Kubernetes"
seoDescription: "Stress testing for your Kubernetes cluster is an important part of ensuring that it is reliable and scalable."
datePublished: Thu Jun 01 2023 10:20:39 GMT+0000 (Coordinated Universal Time)
cuid: cliczjzde03wx2xnvbbzaem3r
slug: stress-command-utilization-in-linux-and-targeting-kubernetes-pods
tags: linux, kubernetes, stress, pods

---

## STRESS

A stress command is a tool for generating workloads and stress tests on a Linux system. It can be used to simulate high CPU, memory, and I/O usage to test the stability and performance of the system.

**Installation:**

I am using the free CentOS version in my lab, so change the commands according to your distribution. Switch to the **root** user from the normal user in the Linux box and follow the below process.

```plaintext
yum install stress -y
```

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1685609611771/08323696-35fd-4239-84c8-eea0c6a802e8.png align="center")

After this, we have to find the number of cores inside the Linux operating system. To fetch the core information from the Linux operating system, use the below command, as shown below:

```plaintext
cat /proc/cpuinfo | grep processor | wc -l
```

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1685609796861/f6319e8d-06d5-4a7e-9439-9b832b87de70.png align="center")

**/proc/cpuinfo** = related to the CPU information.

**grep processor** = search for the processor in the file.

**wc -l** = number of word lines counted in the file.

**Without a timeout,** use the below command, as shown below:

```plaintext
stress --cpu 4 --io 2 --vm 2 --vm-bytes 512M
```

**\--cpu 4** will stress 4 CPUs at 100% load.

**\--io 2** will generate two I/O operations per worker thread.

**\--vm 2** will create two worker threads for stressing the memory.

**\--vm-bytes** 512M will allocate 512 megabytes of memory per worker thread.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1685610529853/c7b521fe-2206-43c3-a0b4-8b27447cb549.png align="center")

To check the command output, go into the duplication of the same server and check the top command as shown below:

```plaintext
top
```

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1685610946463/717137e2-5d75-43bb-a160-2a024f88d3bd.png align="center")

If you want to check with the timeout option, use the below command as shown below:

```plaintext
stress --cpu 4 --io 2 --vm 2 --vm-bytes 512M --timeout 60s
```

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1685611198061/8af4f865-aa52-4f69-9c1a-887bbc9e48e8.webp align="center")

Again, check with the top command, as shown below:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1685611288925/08262a96-9605-43be-8520-7a6a5ce51be6.webp align="center")

Therefore, the stress here is applied successfully in Linux. Now we will continue through the Kubernetes part.

# Kubernetes

Kubernetes is an open-source container **orchestration** system for automating software deployment, scaling, and management.

Kubernetes provides a way to manage containerized applications across multiple hosts. It does this by providing a set of abstractions that allow you to define how your applications should be deployed and scaled. For example, you can use Kubernetes to define how many replicas of an application should be running, or how to distribute those replicas across different hosts.

Kubernetes also provides several features that make it easy to manage your applications. For example, it can automatically restart containers that fail, or scale your applications up or down based on demand.

Install the Kubernetes cluster and follow the below process.

## Why stress in Kubernetes pod?

We can use stress in the Kubernetes pod manifest file to stress test our applications. Stress is a tool that can be used to generate loads on a system. It can be used to test the performance of our applications under load.

Now we will go through how we can create stress in Kubernetes by targeting a specific pod that is hosting an application.

First of all, create the target manifest yaml file as shown below:

```plaintext
apiVersion: v1
kind: Pod
metadata:
  name: nginx-stress
spec:
  containers:
  - name: nginx
    image: ubuntu/nginx
    ports:
    - containerPort: 80
  - name: stress
    image: progrium/stress
    command: ["stress", "--cpu", "2", "--io", "1", "--vm", "1", "--vm-bytes", "128M", "--timeout", "60s"]
```

These manifest files are in YAML format, so try to follow the format as shown above and save them as a file with your comfortable name and the extension **stress.yaml**

```plaintext
kubectl apply -f stress.yaml 
```

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1685612656860/952eb860-23db-4b12-a5e5-7d43375d3bc4.webp align="center")

```plaintext
kubectl get all --all-namespaces
```

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1685612948316/0718d4bf-1c67-4663-86de-4c739c61de7c.webp align="center")

As you can see in the above output, the nginx-stress pod is running successfully, whatever the yaml file we have created. 

Now we are trying to see the pod functionality by using the below command

```plaintext
kubectl describe pod/nginx-stress -n default 
```

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1685613164956/54685e8c-b340-41ec-846f-b989d3f46d5a.webp align="center")

Here we have applied two containers in one pod, like **the Nginx and** **stress** containers in manifest **stress.** After applying the file, the pod is created and running successfully. 

Due to the stress involved in the manifest file, the pod is automatically going down. 

Check if there are any events related to the pod that indicates stress or errors, such as **CrashLoopBackOff, Failed, or Error.** This is an indication that our stress commands are working successfully. 

```plaintext
kubectl get pods
```

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1685613526441/b312b1d1-2851-4391-ba2c-fe49d99658b6.webp align="center")

We have created a stress manifest and nginx file with a timeout of 60 seconds, so it is randomly running for 60 seconds and getting down for 60 seconds, as shown in the above images. If you observe the above output due to stress, it was varying. 

The **stress** here is applied successfully.