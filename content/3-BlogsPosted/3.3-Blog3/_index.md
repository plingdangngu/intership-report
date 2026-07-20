
---
title: "Blog 3"
date: 2026-07-20
weight: 3
chapter: false
pre: " <b> 3.3. </b> "
---

# HOSTING A .NET BLAZOR WEBASSEMBLY APPLICATION ON AMAZON S3 AND AMAZON CLOUDFRONT

Blazor WebAssembly is a Microsoft framework that enables developers to build interactive web applications using C# instead of JavaScript. The application runs directly in the browser through WebAssembly technology. After publishing, it generates a collection of static files, including HTML, CSS, JavaScript, and .NET runtime libraries, making it suitable for deployment on static website hosting services.

The AWS .NET Blog demonstrates how to deploy a Blazor WebAssembly application using Amazon S3 together with Amazon CloudFront. Amazon S3 serves as the storage location for all published application files, while Amazon CloudFront acts as a Content Delivery Network (CDN) that improves website performance by reducing latency and delivering content through AWS Edge Locations worldwide. In addition, the deployment process utilizes Terraform to automate the provisioning of AWS infrastructure.

<div style="text-align: center;">
  <img src="/intership-report/images/Blog/3.3/3.png"
       alt="Blazor WebAssembly deployment architecture"
       width="900">
</div>

<p align="center">
  <i>Figure 1. Deployment architecture of a Blazor WebAssembly application using Amazon S3 and Amazon CloudFront.</i>
</p>

### Main Components of the System

**Blazor WebAssembly**

Blazor WebAssembly is the client-side application developed with .NET. After executing the `dotnet publish` command, the application is compiled into static files that are ready for deployment.

**Amazon S3**

Amazon S3 is used to host all published website files. It provides high scalability, excellent durability, and cost-effective storage for static web applications.

**Amazon CloudFront**

Amazon CloudFront functions as a Content Delivery Network (CDN), distributing website content through Edge Locations around the world. This significantly reduces latency and improves page loading performance.

**Terraform**

Terraform is used to automatically provision AWS resources such as Amazon S3 buckets, CloudFront distributions, and other required infrastructure. By adopting Infrastructure as Code (IaC), deployments become faster, more consistent, and easier to maintain.

#### Benefits of the Architecture

Combining Amazon S3 with Amazon CloudFront provides several advantages:

- Faster website loading through a global CDN.
- Automatic scalability to handle increasing traffic.
- Low deployment cost without managing servers.
- Easy application deployment and updates.
- Infrastructure automation using Terraform.

---

## Deploying the Application on AWS

After publishing the Blazor WebAssembly application, the next step is to deploy it to AWS.

### Step 1: Publish the Application

Use the following command to generate the Release version of the application.

```bash
dotnet publish -c Release
````

After publishing, all website files are generated in the following directory:

```text
bin/Release/net9.0/publish/wwwroot
```

This directory contains the static files that will be uploaded to Amazon S3.

---

### Step 2: Prepare Terraform

Clone the Terraform sample project provided by AWS.

```bash
git clone https://github.com/aws-samples/aws-blazor-hosting.git

cd aws-blazor-hosting
```

Initialize Terraform.

```bash
terraform init
```

---

### Step 3: Deploy the Infrastructure

After configuring the required variables, deploy the infrastructure using:

```bash
terraform apply
```

Terraform automatically provisions the following AWS resources:

* Amazon S3 Bucket
* Amazon CloudFront Distribution
* Origin Access Control (OAC)
* Bucket Policy
* Supporting infrastructure resources

Once the deployment is complete, Terraform returns the CloudFront domain name, which can be used to access the application.

---

### Step 4: Upload the Website to Amazon S3

Synchronize the published website files to Amazon S3 using the AWS CLI.

```bash
aws s3 sync ./publish/wwwroot s3://your-bucket-name
```

After the upload is completed, Amazon CloudFront distributes the website content to end users.

---

### Step 5: Verify the Deployment

Once the CloudFront distribution status changes to **Deployment Completed**, access the CloudFront domain name to verify that the website is running correctly.

Whenever the application is updated, simply publish the project again and synchronize the new files to Amazon S3.

---

## Accelerating Website Performance with CloudFront

CloudFront stores cached copies of website content across multiple Edge Locations worldwide. When users access the website, the requested content is delivered from the nearest Edge Location instead of directly from Amazon S3.

This provides several benefits:

* Reduced latency.
* Faster page loading.
* Lower workload on Amazon S3.
* Improved user experience.

---

## Future Enhancements

After successfully deploying the application, the architecture can be extended with additional AWS services, including:

* AWS Certificate Manager (ACM) for HTTPS support.
* Amazon Route 53 for custom domain names.
* AWS WAF for enhanced security.
* GitHub Actions or AWS CodePipeline for automated deployments.
* Amazon CloudWatch for monitoring and logging.

---

## Advantages and Limitations

### Advantages

* No server management required.
* Low deployment cost.
* High website performance through CloudFront.
* Automatic scalability based on traffic.
* Infrastructure automation with Terraform.
* Easy application updates.

### Limitations

* Suitable only for static websites and client-side applications.
* CloudFront invalidation may be required after updates to refresh cached content immediately.
* Proper configuration of Bucket Policies and Origin Access Control (OAC) is necessary to maintain security.

---

## Conclusion

Deploying a Blazor WebAssembly application on Amazon S3 together with Amazon CloudFront provides an efficient serverless solution for hosting modern web applications. Amazon S3 is responsible for storing the website files, while CloudFront accelerates content delivery through AWS's global CDN network. Furthermore, Terraform automates infrastructure provisioning, making deployments faster, more consistent, and easier to manage.

---

**Reference**


https://aws.amazon.com/vi/blogs/dotnet/host-a-net-blazor-webassembly-app-on-amazon-s3-and-amazon-cloudfront/


