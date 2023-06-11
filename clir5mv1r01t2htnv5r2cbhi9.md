---
title: "Building and Implementing a three tier architecture with AWS  #10WeeksOfCloudOps"
datePublished: Sun Jun 11 2023 08:19:39 GMT+0000 (Coordinated Universal Time)
cuid: clir5mv1r01t2htnv5r2cbhi9
slug: building-and-implementing-a-three-tier-architecture-with-aws-10weeksofcloudops
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1686393115410/7f526f1d-0401-4b81-9027-71bcccfc40a9.webp
tags: aws-three-tier-archirtecture

---

Welcome to my blog. Today we are going to implement a three-tier architecture on the AWS Cloud.

### Introduction:

The three-tier architecture is a very popular way to build out an application. This architecture gives your application the ability to present and interact with it, as also store needed data. It also includes the benefit of having each tier of the architecture be able to run on its infrastructure. Each of the tiers can be developed simultaneously by different teams and can be updated or scaled as needed with little to no impact on the other tiers. This structure is **secure**, **fault-tolerant, and highly available**.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1686392740305/e2bb4653-12bc-4269-bec7-6bb971705871.webp align="center")

A three-tier architecture will include the presentation tier, the application tier, and the database tier. It is the main software architecture for traditional client-server applications.

* **Web Tier** — A presentation or web tier is the user interface and communication layer of an application. This is where the end-user will interact with the application. It will display information and collect information.
    
* **Application Tier** — The logic or application tier is the heart of the architecture. It’s where information is processed. This tier contains all the business logic used to process inputs. It can direct queries, use API calls, process information and commands, make logical decisions, and perform calculations. It moves and processes information to the other layers.
    
* **Database Tier** — And lastly, we have the data or database tier. This is where information will be stored and managed.
    

### Benefits of 3-tier architecture:

1. **Faster development:** Because each tier can be developed simultaneously by different teams, an organization can bring the application to market faster, and programmers can use the latest and best languages and tools for each tier.
    
2. **Improved Scalability:** Any tier can be scaled independently of the others as needed.
    
3. **Improved Reliability:** An outage in one tier is less likely to impact the availability or performance of the other tiers.
    
4. **Improved Security:** Because the presentation and data tiers can’t communicate directly, a well-designed application tier can function as an internal firewall, preventing SQL injections and other malicious exploits.
    

So today, we will build all of the tiers and gain a better understanding of the inner workings of each of those tiers along the way. Here is a diagram of what we are trying to accomplish:

### Architecture:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1686455698548/d83b1478-adb6-40df-8f17-a137773711c9.png align="center")

### Step1: Creating a VPC and subnets

Amazon Virtual Private Cloud (Amazon VPC) is a service provided by Amazon Web Services (AWS) that enables you to create a private virtual network within the AWS cloud. It allows you to define and control a logically isolated section of the AWS cloud where you can launch AWS resources.

**Goto AWS Management console -&gt; Services -&gt; VPC -&gt; Create VPC**

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1686456420175/f8316a20-bc69-4a68-a3cb-7fd0b4d29f5e.png align="center")

* The configuration page for VPC will be displayed. There are two ways to configure VPC.
    
    * **VPC only**: Where you have to configure all the subnets and gateways individually.
        
    * **VPC and more**: Where you have one configuration page and resource map to show all check all the subnets and gateways at once.
        

**When creating the VPC, use the following input:**

**IPv4 CIDR block:** 10.0.0.0/16 -&gt; This is the IP address of the VPC.  
**Tenancy:** Default -&gt; Keep it default so you won't be paying a huge amount to AWS.  
**Availability zones:** 2 -&gt; Specify where you want your two of the AZ to be.  
**Number of public subnets:** 2 -&gt; For our web/presentation tier.  
**Number of private subnets:** 4 -&gt; For our application tier and Database tier.  
**NAT gateways:** In 1 AZ

A NAT (Network Address Translation) Gateway is a managed service provided by Amazon Web Services (AWS) that allows resources within a private subnet in an Amazon Virtual Private Cloud (VPC) to communicate with the internet while keeping them protected from direct inbound traffic. Choose how many of them you want, whether in 1 AZ or 1 per AZ. **(*there is a fee for this, so delete it when you are done)*****VPC endpoints:** None  
**Enable DNS hostnames:** check  
**Enable DNS resolutions:** check

Both of these should be check-marked. It allows you to help DNS Hostname and resolve that into the instance IP of the EC2 you create inside of that VPC.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1686456831782/13b60ad0-fbc5-4a9e-aa97-5ff04c7c8f8c.png align="center")

* **Click on View VPC.**
    
* **Click on subnets -&gt; Choose public subnet -&gt; Actions -&gt; Edit subnet settings -&gt; Enable Auto-assign public IPv4 address.**
    
* **Click “Enable auto-assign IPv4 address” and “Save.” Do this for all 6 of your subnets.**
    

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1686457344540/4b08307a-28bb-4067-90f7-5515fc2c2291.png align="center")

**Click on Route tables -&gt; choose public table -&gt; routes**

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1686457892498/793cfae8-bdab-4536-ac92-3a64ca5bb465.png align="center")

* By using the IP address range 0.0.0.0/0 and attaching the internet gateway to the public subnet, you have set up your network to allow all incoming and outgoing connections with the internet.
    
* Similarly, you can check the route table for private ones and you will find the NAT gateway attached to the private table. Since this method created four different private route tables, you can create only one private route table to include all the subnets and routes with the NAT gateway or two separate route tables for application and data tier respectively.
    
* **Click on Route tables -&gt; choose public table -&gt; subnet association**
    
    You will see the two public subnets associated with that table. Similarly, for other private tables, you will see each of the private subnets associated with each of the private tables.
    
* You will have auto-configured Network Access Control lists. Network Access Control Lists (NACLs) are an AWS service that acts as a virtual firewall for controlling inbound and outbound traffic at the subnet level. Since it is at the subnet level, it has been created already during the configuration of subnets and VPC. We have to provide firewall security at the instance level too.
    

### Step2: Create a web server tier

In this section, we will create our first tier that represents our front-end user interface. We need to create an auto-scaling group of EC2 instances that will host a custom webpage for us.

**Launch an Instance**

Navigate to the EC2 console and **click on Instances** -&gt; **Launch Instance**

Give the instance a name -&gt; **select an AMI** -&gt;**select an instance type** and **key pair**.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1686459934324/608bfdc6-c79c-4d61-be94-82cfaac5d461.png align="center")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1686459966971/9fb21dc6-e4b4-4965-85d0-5fc1baee9c4a.png align="center")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1686460007147/28ad3a51-72ad-4e94-85ed-3b1dc4f3afde.png align="center")

Under the Network settings, you will see that the default VPC is selected.

**Click edit -&gt; Select the VPC -&gt; Created VPC**

**Click subnet -&gt; Choose your subnet**

**Click Auto-assign public IP -&gt; Click Enable**

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1686460225183/45a100ac-9ba3-42c1-a1c2-cd4b05d91c20.png align="center")

Under the Firewall settings, we want to create a web-tier security group that allows inbound internet traffic.

**Select Create a security group -&gt; Select HTTP for type -&gt; Select Anywhere for source**

This will allow all inbound traffic on port 80. I also like to allow HTTPS (port 443) and SSH (port 22).

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1686460589440/b57a37ab-7834-4e11-aa3e-be9664de43c6.png align="center")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1686460609659/1bfe2a57-d15d-4cf1-a10e-09168db8e7dd.png align="center")

Under the Configuration storage settings, you can leave it as is. Open the **Advanced Details** section and scroll to the bottom where you see **user data**.

The following script will install Apache and lead to a custom site that says “**Company Website**.”

```plaintext
#!/bin/bash
  # Use this for your user data (script from top to bottom)
  # install httpd (Linux 2 version)
  yum update -y
  yum install -y httpd
  systemctl start httpd
  systemctl enable httpd
  echo "<h1>Welcome to My Webapp</h1>" > /var/www/html/index.html
```

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1686460931195/d68c35e1-e84a-4284-be7c-3b0e0fe3fc24.png align="center")

We are now ready to click “**Launch instance**.”

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1686461010268/f2bcc8a7-99da-4a65-9401-11dd577a03eb.png align="center")

You should see that your instance is running, but this may take a few moments. Select the running instance and copy and paste the public IPv4 address in a web browser to see if you are met with a custom webpage.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1686461066843/b0a25dbc-3ffc-416e-9d3e-62051dd72b8c.png align="center")

Success! Remember that most web browsers will automatically try to redirect you to the secure HTTP site. If you get an error when trying to access the public IPv4 address, make sure your URL begins with HTTP but not HTTPS.

**Create a Launch Template and Auto Scaling Group**

Ok so now we have a running EC2 instance with a functioning website. We want to attach an auto-scaling group to the instance to increase our availability and reliability. When creating an auto-scaling group, you need to define a launch template that tells EC2 what resources to use to launch the on-demand instances.

In the EC2 console **select Auto Scaling Groups -&gt; Click on Create Auto Scaling group.** Give the auto-scaling group name and **click on Create a Launch template**

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1686461416420/1c60bb5a-17d8-4150-97e8-f4842d307ac5.png align="center")

This will open a new window where you can select the specifics of your launch template. I’m creating a template with the Amazon Linux 2 AMI and t2.micro instance type. Give the launch template a name and description.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1686461558445/18115e0d-acfd-444d-a9a7-337f10c673f4.png align="center")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1686461789123/86bdd208-1363-4f81-a820-009980148c49.png align="center")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1686461807076/8e0751a1-f33c-4f00-8622-ebb7e4bdca91.png align="center")

Select an existing key pair or create a new one. Under Network settings, select the security group that you created earlier for the web tier. You can add additional EBS volumes if desired.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1686461908303/522088c6-04fc-4579-bac4-70f432f2c7f3.png align="center")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1686461952075/67711f32-60e8-4b21-aa22-4b6e9497ec9d.png align="center")

Under Advanced details, include the same user data that you did previously.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1686462059221/de0806ae-3fb6-4bd5-8707-cd85370a0d74.png align="center")

**Click Launch template.**

Ok now, navigate back to the auto-scaling group page, refresh the launch templates, then select the launch template you just created. This launch template is the blueprint the instance will use to create additional instances when it needs to scale up. **Click Next** when ready to proceed.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1686462212599/99b6b7be-d948-4027-8aaa-5c79697b70b4.png align="center")

The next screen asks about network settings and has the default VPC selected. We want our web tier auto-scaling group to use the VPC we created earlier, and scale across the two public subnets that we created. Select the appropriate VPC and subnets and **click Next.**

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1686462362213/04a0cdc3-9d0a-44d4-93e8-9fee87c16c2b.png align="center")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1686462412507/8ade967e-ff5f-4549-b5c5-43c600f143ea.png align="center")

you are now given the option to add a load balancer. A load balancer is a good idea to distribute traffic among the instances.

Since we haven’t created one yet, we will **select Attach to a new load balancer** and **choose Application Load Balancer** because we are using HTTP connections.

Give the load balancer a name and select **Internet-facing** for the scheme. This will allow us to communicate on the Internet, not just internally. Under Network mapping, you should see the VPC that you created and the two public subnets in our auto**\-**scaling group. The load balancer needs a listener and target group. We can see HTTP is selected (port 80) and we will create a new target group.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1686462714228/73001f09-5c7c-485e-9873-c557aaaba3e7.png align="center")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1686462814192/12129fec-e323-4371-9fbc-5759a12c300e.png align="center")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1686462828252/8199f21b-4eba-4e83-a773-65f630a2e698.png align="center")

**Click ELB** under Health checks and **Enable group metrics collection within CloudWatch**. Both of these are optional settings. When ready, **click Next**.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1686462914357/e615aea7-78bd-4908-90e7-670e3cb450d2.png align="center")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1686462939732/130952c4-0258-4ba8-b11a-6098ea567eff.png align="center")

Next, we configure the group size and scaling policies. We want a desired capacity of 1 instance, a minimum capacity of 1 instance, and a maximum capacity of 2 instances.

Selecting the **Target tracking scaling policy** will allow you to apply a policy that resizes your Auto Scale group according to changes in demand. You can set when you want scaling to occur based on a percentage of CPU utilization.

In the below example, I’ve selected the **Target tracking scaling policy** to scale when the average **CPU utilization** reaches **50%**. When you are finished adjusting these settings, **click Next.**

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1686463098569/ca56d271-8790-44f0-a565-dec3c93130e5.png align="center")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1686463128262/8f3083f5-de51-4d53-bab7-58c1f2c55a6c.png align="center")

You have the option to add notifications (SNS topics) or tags, but I’ve skipped both of these. After reviewing your settings for the auto-scaling group, click **Create auto-scaling group**.

**Update Tier 1 Public Route Tables**

Ok at this point, we’ve successfully created our web tier. We have a tier with a VPC, two public subnets with an autoscaling group, a security group, an internet gateway, and user data that will install Apache and create a custom company website landing page.

**The last thing we need to do for the web tier is made sure that the route tables that were automatically created when we created the VPC and subnets are associated correctly.**

Navigate back to the VPC dashboard and on the left side menu **select Route tables**. Select the web tier public route table and view the associated subnets. It should be the two public subnets we created.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1686463477113/adcc7817-4ecc-4d53-84bd-49d6eb25401b.png align="center")

After clicking on the public subnet then you have to verify whether the subnets are attached properly or not.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1686463509892/21d6a683-9904-4ee5-b089-0ae36c4a2dcb.png align="center")

**This looks correct, so our web tier is now complete.**

### Step3: **Creating an Application Tier**

For the application tier, we will use the same VPC as tier 1 as well as 2 of the private subnets that we previously created. We want to again create an EC2 instance with an auto-scaling group of up to 2 and can use the same steps that we did to create our tier 1 instance and auto-scaling group. We are going to change up the security group permissions to limit access from the public. Go ahead and launch an instance using the instructions from earlier.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1686464060722/00b7b3a9-2282-4223-aff1-405dcf27a792.png align="center")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1686464079528/847a97dc-4967-45ca-92ca-887a84d30cac.png align="center")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1686464095794/36212866-d96b-4d0f-8ca3-d89e5e1af513.png align="center")

For Network settings, we want to limit access to the application tier for security purposes. You wouldn’t want any public user to be able to access the application tier or back end, so we will adjust these permissions by creating a new security group.

We want to allow SSH, HTTP, and ICMP traffic from our web-tier security group. The ICMP rule will allow us to ping the application tier from the web tier.

When adding your security groups, choose **Custom** source type, and this will allow you to select your **web tier security group** as the **source**. You do not need to auto-assign public IP for this tier, since it is a private subnet.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1686464282373/62bd5aeb-8127-4cc5-8955-4c0042e7f247.png align="center")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1686464307079/9a18ae73-b957-4c34-8938-8e9a9f9d04a1.png align="center")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1686464323820/118a095e-d37f-48e2-af41-57410f719024.png align="center")

After completing the network settings, go ahead and **launch the instance**.

**<mark>NOTE: In all the protocols add your web-tier security group as a source.</mark>**

**Create a Launch Template and Auto Scaling Group**

Next, let’s create our application tier auto-scaling group. Navigate to the EC2 console and on the left side menu **click Auto Scaling Groups**.

<mark>Make sure you create a new launch template since the security group permissions will be different for the application tier than they were for the web tier</mark>. Use the same process we did before to create an auto-scaling group, and it should look something like this:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1686464830711/1834a105-7c48-47fd-9e81-68e76112077d.png align="center")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1686464848885/4c6a03b0-d278-4ed3-8535-52be67afa676.png align="center")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1686464868541/939eb140-192b-40fb-8d33-c36a9ffc2c5d.png align="center")

Under Network settings, select the application tier security group you created previously.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1686464915880/84cacf44-c483-4afb-87f9-36e359eeaa0a.png align="center")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1686464939590/649c0512-a236-40ad-ad8f-76e737b88261.png align="center")

We aren’t adding any User data to this instance template, so go ahead and **click Create a template**. Now that our application tier launch template is created, let’s go create our application tier auto-scaling group.

Navigate back to the EC2 console in the Create Auto Scaling group page and refresh the launch templates. You should now see the application launch template you just created. Select the application template and **click Next.**

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1686465084122/f1ba734f-5684-4e37-abcb-607f0a5f86a0.png align="center")

Choose the correct VPC and two of your private subnets **(I used private-2-ap-south-2a and private-2-ap-south-2b).**

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1686465236979/6ba65a75-326b-41b3-a568-a8b1fa36ed8c.png align="center")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1686465269251/1fb839e2-1966-4e05-9648-7b3fafaeb9da.png align="center")

After **clicking Next** you are given the option to select a load balancer. <mark>Since the application tier uses private subnets, we can select an internal application load balancer. Ensure that the VPC and private subnets look correct.</mark>

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1686465427737/d3baf13d-7406-47c8-aef8-1466e5b4c63a.png align="center")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1686465446469/620cb9fb-7d22-49f6-a1d8-7551439feaf8.png align="center")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1686465462791/b3fec813-f829-4680-ba4c-a064dbc863c6.png align="center")

You have the option to include health checks and enable monitoring. **Click Next** to proceed.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1686465485384/4046de47-21f9-4572-b196-bc64d7b9d427.png align="center")

Choose how you want your application tier auto-scaling group to scale. The configuration below will always keep one instance running and scale up to 2 when CPU utilization reaches 50%.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1686465620341/b172de71-d4d1-4cd7-9a7f-0c97abb9fb1c.png align="center")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1686465643268/8a5b7735-3005-43da-ab89-bfb451538801.png align="center")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1686465661693/fa0c38e6-4bcf-44ea-b5f6-3b7fb83cfc00.png align="center")

**Click Next.** Add notifications and tags if you want, then review your settings and click **Create Auto Scaling group.**

### Stage 4: Creating a Database tier

We have successfully created two of our three tiers, and now we want to focus on our database. AWS offers several different types of databases, but for this example, we will use a MySQL RDS database.

**Create a DB Subnet Group**

The first thing we have to do is create our subnet groups, so navigate to the RDS console and on the left side menu **click Subnet Groups** and then the orange **Create DB subnet group**. This part can be a bit tricky so bear with me. Start by giving your database subnet group a name and description. Then select the VPC that we have been using for this entire tutorial.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1686466674931/73a91ee5-98df-4033-99df-c30e22ed73a1.png align="center")

Here’s the tricky part. Under Add Subnets, you must first select the availability zones. You need to know which availability zones you used previously to house your third and fourth private subnets.

If you don’t remember, head over to the VPC console, click Subnets on the left side menu, and then select your last two private subnets (make sure not to select the private subnets that you already used in tier 2). On the subnet page, you can see the availability zones used.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1686467093772/6c1487d3-4d0c-4a90-aeb1-e536e9e10ae0.png align="center")

In the above subnets, the last two are my private subnets which I haven't used in any tier. So try to use your private subnets.

Now go back to the RDS Create DB subnet group page and you will see the Add subnets section.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1686467194708/4e5648dd-dcf6-4c85-b3c9-77d020517b6f.png align="center")

Select the availability zones that house the private subnets you’re going to use.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1686467239564/08ba5ba7-47f5-44ce-beef-433a82adf3bb.png align="center")

**Create a MySQL Database**

Your database subnet group is now created, so we can now **click on Databases** on the left side menu of the RDS console, and then **Create Database**. Select **Standard Create** and **MySQL**.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1686467376745/cc6c3e7b-d57f-466b-b2a2-235bfd1316ac.png align="center")

You have the option to create a multi-availability zone cluster that consists of three DB instances (one primary instance and two readable standby instances). This is very useful if a database instance fails, you have two backups ready to launch. For this example, we do not need this feature.

We will use the free tier template. If you were launching this database in Production mode or Dev/Test mode you would have access to the availability and durability deployment options. However, for our example, this is also not needed.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1686467621337/e4cfdb3f-5a1c-4b74-aed7-2c1dd2d1b567.png align="center")

Under the Settings section, give the DB instance a name and create a master username and password for the database. This username and password should be different than your AWS account user login, as it is specific to the database you are creating.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1686467747043/101802df-8c57-44fd-a727-29042ec1258b.png align="center")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1686467765088/afca3b17-f17d-4772-a3ab-a9144c5142c7.png align="center")

**<mark>Make sure you save your username and password. You will need it to connect to your database.</mark>**

Under Instance configuration, the burstable classes option is the only one available for the free tier, thus it is pre-selected. You can adjust your instance type, and I’ve chosen db.t2.micro, which is enough for one VPC. Adjust storage as needed, but for this example, I’ve left it as the default setting.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1686467834881/4c19c489-a028-4706-94a9-ed69dd2ca182.png align="center")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1686467873286/421e13a0-1b5d-45c3-8a37-06e78c3bdc03.png align="center")

The next section is Connectivity and there are a lot of options here. Under Compute resource you can choose if you want to set up a connection to a compute resource for this database.

This option automatically sets up your VPC and related network settings during database creation to enable a secure connection between the EC2 instance and the RDS database.

For my example, I want to set up my network settings manually, so I’ve selected **Don’t Connect to an EC2 compute resource**. The Network type is IPv4, and then you can select the VPC that we have been using for this tutorial. Next, select the DB subnet group that you recently created.

Since this is our company database and we don’t want the public accessing it, we’re going to **select No** for Public Access. Under the VPC security group, we want to create a new security group. Give the security group a name and select a preferred availability zone. You also have the option to create an RDS Proxy, but this does incur additional costs, so we will leave that option unselected.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1686468037329/6cb1a3cf-80fe-40e9-82bf-61f84f9002fb.png align="center")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1686468078340/ccc89801-938c-4dc5-8707-3d585c7d020c.png align="center")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1686468187219/aa523646-5777-4c49-ba07-e1599f0484da.png align="center")

For Database authentication, you have three options. Then you have the option for enhanced monitoring and additional configurations. When ready to proceed, click the **Create Database** button.

**Update Database Tier Security Group**

We created a new security group for the database tier but weren’t allowed to adjust the permissions. Navigate over to the VPC console, select Security Groups on the left side menu, and then find the database tier security group you just created.

You can see that by default, the database security group has an inbound rule to allow MySQL/Aurora traffic on port 3306 from your IP address. Delete this rule.

We want to allow inbound traffic for MySQL from the application tier security group instead. **Click Add rule**, select **MySQL/Aurora** for Type (this will insert port 3306), choose Custom Source, and then select your application tier security group and **Save rules**.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1686468402679/51347c15-9a30-4efe-b554-98eddaa24c19.png align="center")

Edit and save the rule.

The way we can only get into the database from the application tier which enhances security.

Congratulations, you have just created a 3-tier architecture. That was a lot of clicking and steps.

This Project requires a lot of patience, it's a fun project to include in your portfolio.

### **Conclusion**

We have been able to show how we can create a highly available and scalable three-tier web architecture with core AWS services. This architecture consists of a web tier with two public subnets, an application tier with two private subnets, and a database tier with two private subnets. It also provides a detailed guide on creating a VPC, subnets, EC2 instances, security groups, AWS RDS database, auto-scaling, and load balancing. It emphasizes the importance of high availability scalability and fault tolerance in achieving zero downtime or data loss in the event of a disruption.

Thanks for reading.

### **Deleting all resoucres**

1. Delete Database
    
2. Delete Launch Template
    
3. Delete AutoScaling group
    
4. Delete NACL
    
5. Detach Network and delete
    
6. Delete KeyPairs
    
7. Delete Load Balancers
    
8. Delete Target groups
    
9. Delete Nat gateway
    
10. Release the Elastic IP
    
11. Disassociate the Route table and delete the route tables
    
12. Delete Subnets
    
13. Delete Security groups
    
14. Delete VPC
    

### #10WeeksOfCloudOps #aws #Cloud #Architecture

**References:**

[https://www.youtube.com/watch?v=sCBTeMd0Jj4&t=108s](https://www.youtube.com/watch?v=sCBTeMd0Jj4&t=108s)

[What to Consider when Selecting a Region for your Workloads | AWS Architecture Blog (](https://aws.amazon.com/blogs/architecture/what-to-consider-when-selecting-a-region-for-your-workloads/)[amazon.com](http://amazon.com)[)](https://aws.amazon.com/blogs/architecture/what-to-consider-when-selecting-a-region-for-your-workloads/)

[What is Amazon VPC? - Amazon Virtual Private Cloud](https://docs.aws.amazon.com/vpc/latest/userguide/what-is-amazon-vpc.html)

[https://docs.aws.amazon.com/vpc/latest/userguide/VPC\_Route\_Tables.html](https://docs.aws.amazon.com/vpc/latest/userguide/VPC_Route_Tables.html)