---
title: "PART 2: Seamless Integration of Prometheus and Grafana in EKS Cluster for Comprehensive Cluster Metrics and Node.js Application Monitoring."
datePublished: Sun Jul 16 2023 11:13:09 GMT+0000 (Coordinated Universal Time)
cuid: clk5c8u7w0htpfgnvdnpr2moi
slug: part-2-seamless-integration-of-prometheus-and-grafana-in-eks-cluster-for-comprehensive-cluster-metrics-and-nodejs-application-monitoring
tags: implement-monitoring-on-eks-cluster

---

## Architecture:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1689497952596/62282b9e-7205-4b56-92f2-91bb84a3c58e.png align="center")

**Now we will complete the remaining stack setup using the Helm charts. The main components of this stack are Prometheus, Grafana, and Nodejs Applications in EKS Cluster. We will start with the Prometheus.**

## Install Prometheus:

Now install the Prometheus using the helm chart.

Add Prometheus helm chart repository.

```plaintext
$ kubectl create ns prometheus 
####Creating a namespace is best practice so that we will create namespace now 
$ helm repo add stable https://charts.helm.sh/stable
$ helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
$ helm repo update
$ helm search repo prometheus-community
$ helm install stable prometheus-community/kube-prometheus-stack --namespace prometheus 
```

Now we will be able to access the resources in the namespace Prometheus, all pods are running now.

By using the below command we will be able to see the resources in the namespace of Prometheus.

```plaintext
kubectl get all -n prometheus
```

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1689499387186/1ae76633-bed2-4656-ac6a-658943c6f1f2.png align="center")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1689499448672/f870e13a-ec74-495a-ad65-c7770eee1730.png align="center")

Now we want to access the Prometheus dashboard on the web so that we can be able to scrape the metrics and we can add them as a Data Source in the Grafana.

We need to edit the service file **ClusterIP** into **LoadBalancer** so that our Prometheus will be able to run on the web.

```plaintext
kubectl edit svc stable-kube-prometheus-sta-prometheus -n prometheus
```

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1689499905834/23f7a753-2047-40f0-838b-b9131debed5f.png align="center")

Do the changes as **ClusterIP** to **LoadBalancer.** So that we will be able to see the targets in the Prometheus UI as shown in the below image. If you apply the command which is shown below:

```plaintext
kubectl edit svc stable-kube-prometheus-sta-prometheus -n prometheus
```

So that you will be able to see the image which is shown below:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1689500210148/b7ed6b1d-966f-480f-960c-2c45d47fb470.png align="center")

You have to take the load balancer URL with then port 9090. Then you will able to access the Prometheus UI as shown below:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1689500314656/e11efdea-732a-4838-967a-725a5d560499.png align="center")

In the above section, there will be targets available, in that specific section we will be able to see the metrics of every service which is append running in the EKS Cluster.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1689500435234/473c656e-0b7a-4645-b296-12a621b339ac.png align="center")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1689500489701/8a4237db-4877-40c5-bab2-7e1015792f2e.png align="center")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1689500504638/59d0a674-d742-45c7-9314-faff7224ae6d.png align="center")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1689500527383/cd48d340-6215-4f29-b640-4aab771127ab.png align="center")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1689500550594/2a99d33a-3fc5-4ad7-a4d7-11b82c03dd07.png align="center")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1689500563803/1604c1d3-2535-48cb-9e2f-29ab2efc47cd.png align="center")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1689500576031/03352768-cbce-4d05-99bd-5345b734948e.png align="center")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1689500588604/a0ddbc83-421b-49a8-97d8-39aa7b96701a.png align="center")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1689500604768/a26fd1c0-de24-413d-809c-f9b5fc53ee72.png align="center")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1689500625341/769cbca4-cfac-4bda-8a15-7762758d7c51.png align="center")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1689500741928/fa65979d-d6bd-4a31-a109-0145a4e74570.png align="center")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1689500755803/279d0d5f-a82f-4676-b13a-bf172cbc7d21.png align="center")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1689500769750/08af0dde-0bbf-43a5-abb8-c083c5663671.png align="center")

## **Install Grafana:**

Add the ***Grafana*** helm chart repository. Later, Update the helm chart repository.

```plaintext
helm repo add grafana https://grafana.github.io/helm-charts 
helm repo update
```

Now we need to create a Prometheus data source so that Grafana can access the Kubernetes metrics. Create a yaml file **prometheus-datasource.yaml** and save the following data source configuration into it -

```plaintext
datasources:
  datasources.yaml:
    apiVersion: 1
    datasources:
    - name: Prometheus
      type: prometheus
      url: http://prometheus-server.prometheus.svc.cluster.local
      access: proxy
      isDefault: true
```

Create a namespace **grafana**

```plaintext
kubectl create namespace grafana
```

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1689501159856/3d8eb6d5-5a27-42d8-ab5c-e7666d9c3945.png align="center")

Install the Grafana

```plaintext
helm install grafana grafana/grafana \
    --namespace grafana \
    --set persistence.storageClassName="gp2" \
    --set persistence.enabled=true \
    --set adminPassword='EKS!sAWSome' \
    --values prometheus-datasource.yaml \
    --set service.type=LoadBalancer
```

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1689501223531/ae2ed268-54b6-4fbe-b10a-4e7d1a9800b7.png align="center")

Verify the Grafana installation by using the following kubectl command -

```plaintext
kubectl get all -n grafana
```

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1689501352531/3b9ce342-87c4-4f54-a2d2-445ffa191e03.png align="center")

**Copy External IP address** and open it in the browser -

Password you mentioned as **EKS!sAWSome** while creating Grafana

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1689501446079/00075815-2823-47be-8a1a-acb565f5ea77.png align="center")

You can see in the above I have accessed the Grafana using the Loadbalancer and Whatever credentials I have given while installing the Helm charts.

# **Deploy a Node.js application and monitor it on Grafana.**

Now that we have the Docker image, we’ll create a Helm chart so we can deploy the web app to the Kubernetes cluster.

This was the image we pushed into the Dockerhub in the previous Blog.

[Part1 blog](https://shreedhar1998.hashnode.dev/part-1-simplified-dependency-for-eks-cluster-observability-stack-and-dockerizing-the-nodejs-application)

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1689491051428/31c9b1bf-0d48-4701-adfd-f5fccb1974ae.png?auto=compress,format&format=webp align="left")

Create the necessary structure.

```plaintext
$ helm create nodejsapp
In the main user directory follow this command. 
```

The specific command will create the necessary directories for the deployment of the Nodejs. So that we will be able to scrape the metrics to the Prometheus and then we can visualize in the Grafana.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1689502210754/1303720a-6197-40a2-8563-e4d6197a4f28.png align="center")

Go to the sub-directory **nodejsapp** and edit the **values.yaml** file first. Look for the image key first and replace it so it looks like this. Make sure you replace the with your Docker Hub username.

```plaintext
image:
  repository: shreedhar4037/nodejs-cci  ###<username>/<repository name>
  pullPolicy: IfNotPresent
  # Overrides the image tag whose default is the chart appVersion.
  tag: "latest"
```

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1689502921485/47a3eb98-091a-4db9-ae27-a8c75c5935c1.png align="center")

Then look for **podAnnotations: {}** line and replace it so it looks like this.

```plaintext
podAnnotations:
  prometheus.io/scrape: "true"
  prometheus.io/path: "/metrics"
  prometheus.io/port: "3000"
```

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1689502942504/355189da-e53f-4e4a-948a-28ec9f8b40f8.png align="center")

My application is running in the 3000 port. So we can give the same ports to the container and the pod as well. If you want you can change the port here.

Look for the **service** parameter and change it so it looks like this.

```plaintext
service:
  type: LoadBalancer
  port: 80
  targetPort: 3000
  name: nodejsapp-service
```

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1689502964041/e0fa971c-2d45-44e1-9825-149916d17cbc.png align="center")

Look for the resources key and uncomment the defaults.

```plaintext
resources: {}
  # We usually recommend not to specify default resources and to leave this as a conscious
  # choice for the user. This also increases chances charts run on environments with little
  # resources, such as Minikube. If you do want to specify resources, uncomment the following
  # lines, adjust them as necessary, and remove the curly braces after 'resources:'.
limits:
  cpu: 100m
  memory: 128Mi
requests:
  cpu: 100m
  memory: 128Mi
```

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1689503010372/0b12f11b-90cf-45e0-a34c-ef1986993e97.png align="center")

Now we have to get into the directory where the chart folders have located. So we need to edit the **templates/deployment.yaml** file.

Edit the **templates/deployment.yaml** file and find these lines.

```plaintext
ports:
  - name: http
    containerPort: {{ .Values.service.port }}
```

Replace the container port to look like this.

```plaintext
containerPort: 3000
```

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1689503055260/fd6af954-5e6d-4f57-b69b-aca96532915c.png align="center")

Go back to the root of the **nodejsapp** folder and check the chart.

```plaintext
helm install nodejsapp --generate-name
```

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1689503079960/9d4d3055-62a3-4d8a-a334-be847c56502a.png align="center")

Execute the **export SERVICE\_IP** command above and echo the IP. That’s your URL

```plaintext
kubectl get svc 
```

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1689503194358/fffa0b8a-a77b-4d7e-95e5-c88ccca51e54.png align="center")

Copy the load balancer URL and paste it into the web you will be able to see the application which we are running inside the pod of the container.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1689503337084/6d657dc6-17fc-424e-a30c-9b82f8e3e4d8.png align="center")

<mark>The Nodejs Application is running successfully.</mark>

What we care about is the pod. Get the pod running. It’s in the default namespace.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1689503154377/26c2a646-14ef-4ca5-b63c-077f2bbb5df2.png align="center")

Get the annotations. Replace with your pod name.

```plaintext
kubectl get pod nodejsapp-1689442502-648f7b9587-2g8pr -o jsonpath='{.metadata.annotations}'
{"kubernetes.io/psp":"eks.privileged","prometheus.io/path":"/metrics","prometheus.io/port":"3000","prometheus.io/scrape":"true"}
```

# **Import the Grafana dashboard from Grafana Labs.**

Now we have set up everything in terms of Prometheus and Grafana. For the custom Grafana Dashboard, we are going to use the **open-source Grafana** dashboard. For this session, I am going to import a dashboard **6417.**

[Grafana Dashboard](https://grafana.com/grafana/dashboards/)

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1689503914504/c97c7d7e-e155-4736-8387-da1e92b697f5.png align="center")

In the above image, there will be an option for Import. We have to click on that and then we will get an interface as shown below.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1689503980991/603fa9b9-c918-4a76-bf1d-cf44c21bfec0.png align="center")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1689504008553/8006a52b-84c6-42ad-a856-713754aa407e.png align="center")

Click on the Import option then you will get a Dashboard like this as shown in below.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1689504146457/4e4b1e66-3c88-491d-b590-88b3143c250e.png align="center")

Refresh the Grafana dashboard to verify the deployment for getting the Nodejs application in the dashboard.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1689504322485/6335545d-ddc9-48fa-84b8-cff0e770b81d.png align="center")

**I will show you some of the metrics which is related to the Nodejs Application in the Grafana dashboard.**

**Nodejs Deployment usage resources in cluster:**

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1689504558728/1ae70141-306f-4920-b2da-d1208573b985.png align="center")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1689504571546/692e6590-b259-4432-9739-f0c015a71114.png align="center")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1689504584531/156aee50-eeff-4490-9c81-9da456b07701.png align="center")

Now we will see the **EKS Cluster related to Workloads:**

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1689504729234/d8470da0-3c3c-49be-8d95-e528a5d7c388.png align="center")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1689504755819/c2a2a951-09b4-4153-b2ab-4dac9280df7d.png align="center")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1689504771130/2cb3df86-70aa-4d61-ad14-316f5b8cb391.png align="center")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1689504783139/0333ec14-3734-451c-959f-b8753623c36b.png align="center")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1689504816834/3f658569-182b-4416-bfdd-56ef74d3afba.png align="center")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1689504835834/a2cdc828-2bdf-46cb-880d-6a04db454ba9.png align="center")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1689504848653/23e70777-e98b-48fe-a795-46878de11a54.png align="center")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1689504856080/f6dcb688-817b-4457-b865-741888cfa927.png align="center")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1689504871895/5936b2c0-8a0e-42ad-babc-0333faf4daf6.png align="center")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1689504884530/15be19f6-f907-43fd-a6c7-cbece4d53f9a.png align="center")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1689504898954/f8241549-d2f2-4b3d-8575-c8b90128659b.png align="center")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1689504906873/bfdbcb7b-ffb3-4a4a-b0fc-c6bc68037496.png align="center")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1689504917073/2f1c8d38-f590-419c-923a-d8336da8b306.png align="center")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1689504957963/b3cd45f9-270f-40d0-8ad7-45c2b65aea50.png align="center")

# **Clean Up**

In this stage, you're going to clean up and remove all resources which we created during the session. So that it will not be charged to you afterward.

1. Delete the EKS cluster with the following command.
    

```plaintext
eksctl delete cluster --name <clustername>
```

1. Delete EC2 Instance.
    

### **References:**

[**https://prometheus.io/**](https://prometheus.io/)

[**https://grafana.com/**](https://grafana.com/)

[**https://kubernetes.io/**](https://kubernetes.io/)

Follow this blog channel for further interesting topics in the

### #10weeksofcloudops

[**Follow me on Linkedin**](https://www.linkedin.com/in/sridhar-modalavalasa-12b492173/)

[**Follow me on GitHub**](https://github.com/sridhar-modalavalasa)

# **Thank you:**

I conclude this exercise will help you to understand the concepts of using Kubernetes metrics monitoring using Prometheus and Grafana dashboards.

***Thanks for reading to the end; I hope you gained some knowledge.***