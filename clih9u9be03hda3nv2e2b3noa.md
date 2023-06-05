---
title: "Deployment of the static website using AWS CodePipeline, S3, and GitHub."
seoTitle: "Static Website in AWS"
datePublished: Sun Jun 04 2023 10:19:39 GMT+0000 (Coordinated Universal Time)
cuid: clih9u9be03hda3nv2e2b3noa
slug: deployment-of-the-static-website-using-aws-codepipeline-s3-and-github
tags: deployment-of-static-website-in-aws

---

# #10WeeksOfCloudOps

### Objective:

To deploy our website via AWS CodePipeline. The diagram below shows the workflow that we need to achieve to get the job accomplished.

### Architecture:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1685872538531/9b8aa9d1-1892-4f69-9539-97fa5f75a464.png align="center")

### **Prerequisite:**

* **GitHub Repository with HTML and CSS**
    
* **S3 bucket Creation**
    
* **Configuring CloudFront**
    
* **Implementing CI/CD through AWS CodePipeline**
    
* **Time to Learn and Build**
    

### **Step1: Upload/Push your HTML and CSS files to the GitHub Repo**

The first step is to push your website HTML and other files to your repo. For this project, I have created HTML and CSS files to be deployed.

[GitHub Repo](https://github.com/sridhar-modalavalasa/Staticwebsite)

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1685857413164/92936bca-3fe9-4e01-a852-9add2da69da7.png align="center")

### **Step2: S3 Bucket Creation**

An S3 bucket is a storage container in Amazon Simple Storage Service (S3). S3 is a web service that provides object storage, which means that data is stored in distinct units called objects instead of files.

Each bucket has a unique name and can be located in a specific region. S3 buckets are similar to file folders and can be used to store, retrieve, back up, and access objects. Each object has three main components:

* **Object content or data:** This is the actual data that is stored in the bucket.
    
* **Object key:** This is a unique identifier for the object.
    
* **Object metadata:** This is additional information about the object, such as its name, size, and creation date.
    

**Goto Services -&gt; S3 -&gt; Create Bucket**

**Bucket Name:** Keep the bucket name unique and for now it should be your domain name. For example\*\*,\*\* [learnwithshree.infinityfreeapp.com](http://learnwithshree.infinityfreeapp.com) my domain name is ( you can exclude or include www but later it should be the same in Route 53 ) so my bucket name will be [learnwithshree.infinityfreeapp.com](http://learnwithshree.infinityfreeapp.com)

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1685855028156/37c8baf0-c76a-4c04-95de-5f4bbfca8605.png align="center")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1685855084080/5f621cc2-9e9a-416c-a85b-c690fffde278.png align="center")

**AWS Region:** This is the region where you will like to store the buckets.

**Block all public access:** Most people prefer keeping it open while serving static websites but we will be using CloudFront and SSL/TLS certificate to serve the website content. so we will block public access in our case.

**Keep the other configuration default for now.**

**Now your bucket will be created.** Go inside the bucket and upload the file. Prefer keeping the main html file as index.html as an S3 searches file named with index to serve. If you have CSS and simple js inside the folder, you can upload that too.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1685857656027/267fc6ea-2cfd-48b9-b688-a0b141f9841b.png align="left")

After the files have been uploaded, you will see something like the above image. Here, I have only the index.html file, website.css I have created as part of this project.

You can see the properties page, Click on that and you will find the property to host a static website at the bottom.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1685858330253/8f55c441-06ef-4517-ad6a-634d823a60ea.png align="center")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1685858395455/5cd02aef-7ac0-4d6d-acfc-78f2fa5e2b0f.png align="center")

The edit button brings you to this interface. You now enable static website hosting. There are two options in hosting type viz. Host a static website which refers to hosting your static website directly and Redirect requests for an object which redirects one bucket to another. Give the index document name an index.html which is your main HTML file. You also have the optional choice to provide an error document which is error.html that can show some beautiful error page instead of AWS's default ugly error page.

After you have enabled static web hosting, you will get the website endpoint at the bottom of the properties tab. You just click on that and you will see your website.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1685858720275/911ce787-33e2-4dc2-9363-79b9d62beb04.png align="center")

By clicking on the above link on your website you will get an error as shown in the below image. This is due to the policy is not attached to the bucket.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1685858799207/1b7a7149-af99-47e6-a339-d3ac1fb9b7ce.png align="center")

It might be because we have blocked public access and also we have forgotten to provide a bucket policy to our bucket. So now let's visit the permission tab inside the bucket.

Go to your bucket and select **Permissions** and look for **Block public access** and click **Edit**

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1685858993488/84e118d4-a5fd-46af-8928-a40e9491b035.png align="center")

Uncheck **Block all public access** and click **Save changes**. You will get a prompt to confirm save changes so enter confirm and save.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1685859175567/934965a0-5a7e-4bda-911c-e3ddba05940e.png align="center")

You have to add a bucket policy. Your resource will be in **bucket-&gt;properties-&gt; Amazon Resource Name (ARN).**

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1685858927352/221defe1-0913-414a-82f6-4fa35899c5a0.png align="center")

In this way, we need to add the bucket policy as shown below and get the policy from the website which is shown in the above image.

[PolicyUrl](https://docs.amazonaws.cn/en_us/AmazonS3/latest/userguide/HostingWebsiteOnS3Setup.html)

```plaintext
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "PublicReadGetObject",
            "Effect": "Allow",
            "Principal": "*",
            "Action": [
                "s3:GetObject"
            ],
            "Resource": [
                "arn:aws-cn:s3:::Bucket-Name/*"
            ]
        }
    ]
}
```

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1685859611883/1af5393b-1ca5-4eb1-987d-49c812b1cf6d.png align="center")

Again you have to go to the **bucket -&gt; properties -&gt; static website** and click on the URL which is shown below.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1685859883189/97a2921a-0253-4109-a2ef-614bfedb9a8f.png align="center")

After clicking on the above link which is shown in the above image you will get your interface which you have written in the HTML and CSS scripts as shown below. Copy the URL and open it in the browser and your website should work now like mine.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1685859992279/0291c981-18c8-444c-afe6-2d6c0211c3e5.png align="center")

### **Step 3: Configuring Cloudfront**

SSL (Secure Sockets Layer) and TLS (Transport Layer Security) are cryptographic protocols designed to provide secure communication over the Internet. They establish an encrypted connection between a client (such as a web browser) and a server, ensuring that data transmitted between them remains private and secure.

The custom domain is now pointing to our website but we are not able to secure it. For securing and caching the content faster and in a more secure way, we will be using CloudFront.

**Goto Services -&gt; CloudFront -&gt; Create Distribution**

Creating a distribution is a bit long but not so complicated process. Let's walk through it. (The option that I will not mention, you can leave them default).

Click on **Create a CloudFront distribution**

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1685862102639/9f973ef3-b259-4117-b638-657ddacc0dde.png align="center")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1685863518099/11812e2f-6d6f-4c6d-bca7-8fcb4e17a3b6.png align="center")

Disable cache as if you will not do it then it may take long time to update changes on your website.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1685863797280/7daed71e-3fa2-404a-a73b-b58b71622df6.webp align="center")

On **Standard logging** and creating distribution

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1685864113533/e6d5ebd9-2b2c-4972-bb66-6f2c235f84f1.png align="center")

After creating the distribution you can see the Distribution domain name, copy, and view it in the browser. This should show your website.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1685864522396/3de8746c-caec-40b7-90cd-7102adcacd2b.png align="center")

### **Step 4: Implementing CI/CD through AWS CodePipeline**

CI/CD stands for Continuous Integration and Continuous Deployment (or Continuous Delivery). It is a set of practices and processes used in software development to automate the building, testing, and deployment of applications.

We will be using AWS Codepipeline and GitHub for the CI-CD process.

### **a.Connect GitHub Account to CodePipeline**

Navigate to the CodePipeline service and “**Create pipeline**”.

Name your pipeline and select “**New** **service role**”. Leave the rest of the settings at default, then click “**Next**”.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1685867270514/48939333-b30c-4027-aa65-b879d1c8a4a7.png align="center")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1685867469763/805ce2ea-32a5-4dfb-984f-7a7fed69317c.png align="center")

We need to connect to GitHub in this pipeline for CI/CD automation.

For the source stage, select “**GitHub (Version 2)**” as our source provider.

We can now connect to our GitHub by selecting “**Connect to GitHub**”. Name the connection, then “**Connect to GitHub**”. You’ll need to sign in to your GitHub account to authorize the connection.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1685867605814/b1c4fa0a-52c1-458a-b9d2-138dacd867d5.png align="center")

Click “**Install a new app**”, select your Tourism Website repository, click “**Install**”, then click “**Connect**”.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1685867826197/2f97ec28-89be-42ee-a0a8-6cab7b9d54e8.png align="center")

After this, we need to give access by authorizing credentials as shown in the below image.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1685867912923/88e11329-c396-4611-b72b-2ddb081436e3.png align="center")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1685868072903/6d56a0f6-81c6-4e37-8d22-cd857698b27d.png align="center")

### **b.Configure CodePipeline and deploy CI/CD pipeline**

Proceed by adding the “**Repository name**” and the “**Branch name**”. All other settings can remain at default, then click “**Next**”.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1685868178816/7935d3d2-8a36-4d0f-929b-4e83a2a504b6.png align="center")

If you scroll down a little bit there will be options as shown below.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1685868251138/26b1de13-df57-4109-ba92-bdd5c7415167.png align="center")

We will skip the build stage by clicking **“Skip build stage”**.

For the “**Add deploy stage**”, select “**S3**” for the deploy provider, then the region your S3 bucket was created in, then your Tourism Website bucket. Ensure to select “**Extract file before deploy**”, then click “**Next**”.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1685868386356/cafd54f1-9dcd-446b-915e-698e97397fab.png align="center")

Review the pipeline, then proceed to “**Create pipeline**”. Wait for the pipeline to be created.

### **c.Delete all previous files in the S3 bucket**

All previous configurations with CloudFront will remain, however, since your code is now being hosted in GitHub, we will delete the previous Tourism Website files in your S3 bucket.

Head to your AWS Management console and navigate to the S3 service, select “**Buckets**”, then select the bucket hosting your Tourism Website files. Select all the files, then proceed to delete them.

**Success**, If everything was done correctly, you should see a success message, as shown below.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1685868826171/f0d48b0b-2833-4d58-ab35-2a11e7506986.png align="center")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1685868891248/bcef594e-cc96-4f9b-95ee-9457c01df154.png align="center")

### Congratulations!

You have just created a CI/CD pipeline for your Tourism Website, using AWS CodePipeline, GitHub, and Amazon S3!

If you head to your S3 bucket that host’s your Tourism Website files, you should see the newly updated files, including the “[**READMe.md**](http://READMe.md)” file. This verifies that our CI/CD pipeline was able to make updates aligning with our GitHub repo.

We can now **proceed to the final Step: Verifying our CI/CD pipeline!**

### **Verify functionality of CI/CD Pipeline**

Before we can fully verify the functionality of the CI/CD pipeline, we first need to verify that we can view our Tourism Website from our custom domain in our browser.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1685869052842/682375a4-230a-4ac8-b3e9-cb7c499764d0.png align="center")

Great! We are able to view our Tourism Website!

Now, we will make an update to the code in GitHub to verify that the pipeline is triggered and the CI/CD pipeline is functional. This will be as easy as simply making a change to the “[**README.md**](http://README.md)” file since any change to the files should trigger the workflow.

After making changes, we can head back to CodePipeline. If you select your pipeline, you should notice that the pipeline has been triggered just recently.

Also, if we click “**History**” on the left pane of the CodePipeline dashboard, we can view the “**Executing history**” and see that it has been triggered twice. The initial, was when we set up the pipeline, and the second time was when we made changes to the “[**README.md**](http://README.md)” file.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1685870901720/9b8ca560-4567-4c38-b363-f4fc129b303d.png align="center")

I have updated the tourism button description as **View Info** from **Get Started** as you can verify on the above page.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1685871098785/4f87b166-1abd-455f-b10b-4f27b886c484.png align="center")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1685871206876/abc4d92d-d2e0-40fd-9521-3be22a068283.png align="center")

If you click on the View Info you will get a description as mentioned in the below.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1685871242145/1e9dfe97-929a-43da-8c38-684a090d861d.png align="center")

### **Success!**

If everything was done correctly, you’ve just successfully created a CI/CD pipeline to automate the deployment of your static Website using AWS CodePipeline, GitHub, and Amazon S3!

If you’ve got this far, **thanks for reading!** I hope it was worthwhile for you.