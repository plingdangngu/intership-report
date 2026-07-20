
---
title: "Blog 2"
date: 2026-07-20
weight: 2
chapter: false
pre: " <b> 3.2. </b> "
---

# HOSTING A .NET BLAZOR WEBASSEMBLY APPLICATION ON AMAZON S3 AND AMAZON CLOUDFRONT

Blazor WebAssembly is a framework within the .NET ecosystem that enables developers to build interactive web applications using C#. Unlike traditional web applications that require a server to process the user interface, Blazor WebAssembly can run directly in the user's browser through WebAssembly technology.

After the application is compiled using the `dotnet publish` command, the output contains static files such as HTML, CSS, JavaScript, WebAssembly files, and the required libraries. Because the application does not require a continuously running .NET server, it can be deployed as a static website on Amazon S3.

To improve access speed, support HTTPS, and distribute content to users across different geographical locations, Amazon CloudFront can be used as a Content Delivery Network. CloudFront receives requests from users, caches content at edge locations, and retrieves data from Amazon S3 when required.

In addition, the infrastructure can be managed using Terraform. Infrastructure as Code makes creating resources, updating configurations, and redeploying the system more consistent while reducing errors caused by manual configuration.

<div style="text-align: center;">
  <img src="/intership-report/images/Blog/3.2/1.png"
       alt="Architecture for hosting Blazor WebAssembly on Amazon S3 and Amazon CloudFront"
       width="900">
</div>

<p align="center">
  <i>Figure 1. Architecture for hosting a .NET Blazor WebAssembly application using Amazon S3 and Amazon CloudFront.</i>
</p>

### Architecture Overview

The Blazor WebAssembly hosting architecture consists of three main components:

1. **Blazor WebAssembly**: The .NET application is compiled into static files.
2. **Amazon S3**: Stores all compiled application files.
3. **Amazon CloudFront**: Distributes content to users through AWS edge locations.

In this architecture, the Amazon S3 bucket is configured as private. Users do not access the S3 bucket directly. Instead, all requests are sent through CloudFront.

CloudFront uses Origin Access Control to sign requests using AWS Signature Version 4 before accessing content stored in Amazon S3. The S3 bucket policy only permits a specified CloudFront distribution to read objects from the bucket.

This mechanism protects the application's content more effectively than making the S3 bucket publicly accessible.

### Benefits of Combining Amazon S3 and Amazon CloudFront

Hosting a Blazor WebAssembly application using Amazon S3 and CloudFront provides several operational benefits.

**No server management**

A Blazor WebAssembly application is compiled into static files, so there is no need to maintain a dedicated web server or .NET server. As a result, administrators do not need to update an operating system, install security patches, or manage server resources.

**Scalability**

Amazon S3 can store a large number of objects, while CloudFront can handle increased traffic through its global network of edge locations. There is no need to configure Auto Scaling or predict the number of users in advance.

**Faster access**

CloudFront caches content at edge locations close to users. When an object is already available in the cache, CloudFront can return it directly without sending another request to Amazon S3, thereby reducing application loading latency.

**HTTPS support**

CloudFront provides a default HTTPS endpoint. When a custom domain is used, CloudFront can be integrated with AWS Certificate Manager to create and manage an SSL/TLS certificate.

**Cost optimization**

Because no server runs continuously, there are no compute instance costs. The primary costs come from Amazon S3 storage, requests, and the amount of data transferred from CloudFront to users.

### Important Design Decisions

#### Using Origin Access Control

Origin Access Control, or OAC, is the mechanism CloudFront uses to securely access an Amazon S3 origin.

Instead of making the S3 bucket public, CloudFront signs every request using AWS Signature Version 4. The bucket policy allows only the specified CloudFront distribution to access the stored objects.

The main benefits of OAC include:

- The S3 bucket does not need to be publicly accessible.
- Only the specified CloudFront distribution can read the application's files.
- Requests from CloudFront to Amazon S3 are signed using AWS Signature Version 4.
- Users cannot bypass CloudFront and directly access the S3 origin.
- The configuration follows the recommended modern approach for securely distributing content from Amazon S3.

#### Disabling S3 Static Website Hosting

In this architecture, Amazon S3 is used only for object storage. The S3 Static Website Hosting feature does not need to be enabled because CloudFront is the only public entry point.

The bucket blocks all public access, and CloudFront connects to the regional Amazon S3 endpoint through Origin Access Control.

This configuration provides the following benefits:

- Prevents users from accessing the S3 bucket directly.
- Ensures that users access the application through CloudFront HTTPS.
- Applies consistent caching rules.
- Simplifies custom domain and certificate configuration.

#### Separating Cache Policies

Not all files in a Blazor WebAssembly application should use the same cache duration.

Files inside the `_framework` directory usually contain hash values in their filenames. When their content changes, their filenames also change. Therefore, these files can be safely cached for a long period.

In contrast, `index.html` references the files belonging to the current application version. If this file is cached for too long, users may continue loading an outdated version after a new release is deployed.

For this reason, the architecture uses different caching strategies:

- `_framework/*`: Cached for one year and marked as `immutable`.
- `index.html`: Not cached so that users always receive the latest version.
- CSS files, images, and libraries: Can be cached for a long period when their filenames are versioned.

### Handling Client-side Routing

Blazor WebAssembly is a Single Page Application. Navigation between pages is handled in the browser by the Blazor Router.

For example, when a user accesses:

```text
https://example.com/counter
```

The browser sends a request for the `/counter` path. However, there is no object named `counter` in the Amazon S3 bucket.

Because the S3 bucket is private, the request may return a `403 Forbidden` response. In other configurations, the request may return `404 Not Found`.

To resolve this issue, CloudFront is configured with Custom Error Responses:

- Convert a `403` response to `/index.html`.
- Convert a `404` response to `/index.html`.
- Return the page with an HTTP `200` status code.

After `index.html` is loaded, the Blazor Router reads the current URL and displays the corresponding component.

This solution is not limited to Blazor WebAssembly. It can also be applied to other Single Page Application frameworks such as React, Angular, Vue, and Svelte.

---

## Environment Preparation

Before beginning the deployment, the following tools must be installed and configured:

- .NET SDK version 10.0 or later.
- Terraform version 1.6 or later.
- AWS Command Line Interface version 2.
- Git for downloading and managing source code.
- An AWS account with permission to create Amazon S3 buckets and CloudFront distributions.

Check the installed .NET SDK version:

```bash
dotnet --version
```

Check the Terraform version:

```bash
terraform --version
```

Check the AWS CLI version:

```bash
aws --version
```

Configure the AWS CLI using an access key or another supported authentication method:

```bash
aws configure
```

Enter the required information:

```text
AWS Access Key ID
AWS Secret Access Key
Default region name
Default output format
```

The IAM user or IAM role used for deployment should have permission to manage resources such as:

- Amazon S3 buckets.
- S3 bucket policies.
- S3 Public Access Block settings.
- Amazon CloudFront distributions.
- CloudFront Origin Access Control.
- CloudFront cache policies.
- CloudFront invalidations.
- AWS Certificate Manager and Amazon Route 53 when a custom domain is used.

---

## Creating the Blazor WebAssembly Application

### Step 1: Create the Project

Use the following command to create a new .NET 10 Blazor WebAssembly project:

```bash
dotnet new blazorwasm \
    -o src/BlazorApp \
    --framework net10.0
```

The project will have a directory structure similar to the following:

```text
src/
└── BlazorApp/
    ├── Layout/
    ├── Pages/
    ├── Properties/
    ├── wwwroot/
    ├── App.razor
    ├── Program.cs
    └── BlazorApp.csproj
```

The default template provides sample pages such as:

- Home.
- Counter.
- Weather.

These pages can be used to test navigation, interface updates, and data loading within the application.

### Step 2: Run the Application Locally

Before deploying the application to AWS, test it in the local environment:

```bash
dotnet run --project src/BlazorApp
```

After the application starts, the terminal displays a local address similar to:

```text
https://localhost:5001
```

Open this address in a browser and test the Home, Counter, and Weather pages.

### Step 3: Publish the Application

Use the following command to produce the static files required for deployment:

```bash
dotnet publish src/BlazorApp/BlazorApp.csproj \
    -c Release \
    -o publish \
    --nologo
```

After publishing is complete, the files that need to be deployed are located in:

```text
publish/wwwroot/
```

This directory contains files similar to:

```text
publish/wwwroot/
├── _framework/
├── css/
├── sample-data/
├── index.html
├── favicon.png
└── icon-512.png
```

All content inside the `wwwroot` directory can be uploaded as objects to Amazon S3.

---

## Building the Infrastructure with Terraform

Terraform is used to automatically create the Amazon S3 bucket, Origin Access Control, cache policies, and CloudFront distribution.

The Terraform directory can be organized as follows:

```text
infra/
├── main.tf
├── cloudfront.tf
├── variables.tf
├── outputs.tf
└── terraform.tfvars
```

The files serve the following purposes:

- `main.tf`: Defines the S3 bucket, versioning, and bucket policy.
- `cloudfront.tf`: Defines CloudFront, OAC, and cache policies.
- `variables.tf`: Defines input variables.
- `outputs.tf`: Returns deployment information.
- `terraform.tfvars`: Provides values for the variables.

### Step 1: Define the AWS Provider

Add the AWS Provider configuration to `main.tf`:

```hcl
terraform {
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 6.0"
    }
  }

  required_version = ">= 1.6.0"
}

provider "aws" {
  region = var.aws_region
}
```

### Step 2: Create the Amazon S3 Bucket

Add the following configuration to create the S3 bucket:

```hcl
resource "aws_s3_bucket" "blazor_app" {
  bucket = var.bucket_name

  tags = {
    Environment = var.environment
    ManagedBy   = "terraform"
  }
}
```

The S3 bucket name must be globally unique. Therefore, replace the `bucket_name` value with a unique name.

### Step 3: Block Public Access

Because the S3 bucket should only be accessed through CloudFront, configure S3 Public Access Block:

```hcl
resource "aws_s3_bucket_public_access_block" "blazor_app" {
  bucket = aws_s3_bucket.blazor_app.id

  block_public_acls       = true
  block_public_policy     = true
  ignore_public_acls      = true
  restrict_public_buckets = true
}
```

This configuration prevents ACLs or bucket policies from accidentally making the bucket publicly accessible.

### Step 4: Enable Versioning

S3 Versioning preserves multiple versions of the same object:

```hcl
resource "aws_s3_bucket_versioning" "blazor_app" {
  bucket = aws_s3_bucket.blazor_app.id

  versioning_configuration {
    status = "Enabled"
  }
}
```

If a file is overwritten or accidentally deleted, an earlier version can be restored.

### Step 5: Create Origin Access Control

In `cloudfront.tf`, define Origin Access Control:

```hcl
resource "aws_cloudfront_origin_access_control" "blazor_app" {
  name                              = "${var.bucket_name}-oac"
  description                       = "OAC for Blazor WebAssembly application"
  origin_access_control_origin_type = "s3"
  signing_behavior                  = "always"
  signing_protocol                  = "sigv4"
}
```

The `signing_behavior = "always"` setting requires CloudFront to sign every request sent to Amazon S3 using AWS Signature Version 4.

### Step 6: Create a Long-term Cache Policy

Create a cache policy for files with content hashes in their filenames:

```hcl
resource "aws_cloudfront_cache_policy" "immutable_assets" {
  name        = "${var.bucket_name}-immutable-assets"
  min_ttl     = 31536000
  default_ttl = 31536000
  max_ttl     = 31536000

  parameters_in_cache_key_and_forwarded_to_origin {
    cookies_config {
      cookie_behavior = "none"
    }

    headers_config {
      header_behavior = "none"
    }

    query_strings_config {
      query_string_behavior = "none"
    }

    enable_accept_encoding_brotli = true
    enable_accept_encoding_gzip   = true
  }
}
```

The value `31536000` represents one year in seconds.

### Step 7: Create a No-cache Policy

Create a cache policy for `index.html` and paths that do not match the `_framework` directory:

```hcl
resource "aws_cloudfront_cache_policy" "no_cache" {
  name        = "${var.bucket_name}-no-cache"
  min_ttl     = 0
  default_ttl = 0
  max_ttl     = 0

  parameters_in_cache_key_and_forwarded_to_origin {
    cookies_config {
      cookie_behavior = "none"
    }

    headers_config {
      header_behavior = "none"
    }

    query_strings_config {
      query_string_behavior = "none"
    }

    enable_accept_encoding_brotli = false
    enable_accept_encoding_gzip   = false
  }
}
```

This policy ensures that CloudFront retrieves the latest version of `index.html`.

### Step 8: Create the CloudFront Distribution

Define the CloudFront distribution:

```hcl
resource "aws_cloudfront_distribution" "blazor_app" {
  enabled             = true
  is_ipv6_enabled     = true
  default_root_object = "index.html"
  price_class         = var.cloudfront_price_class

  origin {
    domain_name              = aws_s3_bucket.blazor_app.bucket_regional_domain_name
    origin_id                = "s3-blazor-app"
    origin_access_control_id = aws_cloudfront_origin_access_control.blazor_app.id
  }

  default_cache_behavior {
    target_origin_id       = "s3-blazor-app"
    viewer_protocol_policy = "redirect-to-https"
    allowed_methods        = ["GET", "HEAD", "OPTIONS"]
    cached_methods         = ["GET", "HEAD"]
    compress               = true
    cache_policy_id        = aws_cloudfront_cache_policy.no_cache.id
  }

  ordered_cache_behavior {
    path_pattern           = "_framework/*"
    target_origin_id       = "s3-blazor-app"
    viewer_protocol_policy = "redirect-to-https"
    allowed_methods        = ["GET", "HEAD", "OPTIONS"]
    cached_methods         = ["GET", "HEAD"]
    compress               = true
    cache_policy_id        = aws_cloudfront_cache_policy.immutable_assets.id
  }

  custom_error_response {
    error_code            = 403
    response_code         = 200
    response_page_path    = "/index.html"
    error_caching_min_ttl = 0
  }

  custom_error_response {
    error_code            = 404
    response_code         = 200
    response_page_path    = "/index.html"
    error_caching_min_ttl = 0
  }

  restrictions {
    geo_restriction {
      restriction_type = "none"
    }
  }

  viewer_certificate {
    cloudfront_default_certificate = true
  }
}
```

Important settings include:

- `default_root_object = "index.html"`: Loads the main page when the domain is accessed.
- `redirect-to-https`: Automatically redirects HTTP requests to HTTPS.
- `_framework/*`: Applies long-term caching to Blazor framework files.
- `403` and `404`: Redirect requests to `index.html` to support client-side routing.
- `compress = true`: Enables CloudFront content compression.

### Step 9: Configure the S3 Bucket Policy

The bucket policy allows only the newly created CloudFront distribution to read objects:

```hcl
data "aws_iam_policy_document" "s3_cloudfront_read" {
  statement {
    actions = ["s3:GetObject"]

    resources = [
      "${aws_s3_bucket.blazor_app.arn}/*"
    ]

    principals {
      type        = "Service"
      identifiers = ["cloudfront.amazonaws.com"]
    }

    condition {
      test     = "StringEquals"
      variable = "AWS:SourceArn"
      values   = [aws_cloudfront_distribution.blazor_app.arn]
    }
  }
}

resource "aws_s3_bucket_policy" "blazor_app" {
  bucket = aws_s3_bucket.blazor_app.id
  policy = data.aws_iam_policy_document.s3_cloudfront_read.json

  depends_on = [
    aws_s3_bucket_public_access_block.blazor_app
  ]
}
```

The `AWS:SourceArn` condition limits access to the specific CloudFront distribution used by the application.

### Step 10: Define Variables

Add the following variables to `variables.tf`:

```hcl
variable "aws_region" {
  description = "AWS Region"
  type        = string
  default     = "us-west-2"
}

variable "bucket_name" {
  description = "S3 bucket name"
  type        = string
}

variable "environment" {
  description = "Deployment environment"
  type        = string
  default     = "prod"
}

variable "cloudfront_price_class" {
  description = "CloudFront price class"
  type        = string
  default     = "PriceClass_100"
}
```

### Step 11: Provide Variable Values

Create a `terraform.tfvars` file:

```hcl
aws_region             = "us-west-2"
bucket_name            = "my-blazor-wasm-app"
environment            = "prod"
cloudfront_price_class = "PriceClass_100"
```

Replace `my-blazor-wasm-app` with an unused and globally unique S3 bucket name.

### Step 12: Define Outputs

Add the following configuration to `outputs.tf`:

```hcl
output "s3_bucket_name" {
  value = aws_s3_bucket.blazor_app.id
}

output "cloudfront_distribution_id" {
  value = aws_cloudfront_distribution.blazor_app.id
}

output "app_url" {
  value = "https://${aws_cloudfront_distribution.blazor_app.domain_name}"
}
```

These outputs are used when uploading the application and clearing the CloudFront cache.

---

## Deploying the Infrastructure

Move into the Terraform directory:

```bash
cd infra
```

Initialize Terraform:

```bash
terraform init
```

Review the resources that Terraform will create:

```bash
terraform plan
```

Deploy the infrastructure:

```bash
terraform apply
```

Enter:

```text
yes
```

After the deployment is complete, Terraform returns information similar to:

```text
Outputs:

s3_bucket_name               = "my-blazor-wasm-app"
cloudfront_distribution_id   = "EXAMPLE123456"
app_url                      = "https://example.cloudfront.net"
```

CloudFront may require some time to distribute the configuration to its edge locations before the application becomes available.

<div style="text-align: center;">
  <img src="/intership-report/images/Blog/3.2/2.png"
       alt="Terraform infrastructure deployment result"
       width="900">
</div>

<p align="center">
  <i>Figure 2. Amazon S3 bucket and CloudFront distribution created using Terraform.</i>
</p>

---

## Uploading the Application to Amazon S3

After the infrastructure has been created, build the application and upload the files from `publish/wwwroot` to Amazon S3.

### Uploading Standard Files

The following command can be used to upload the website:

```bash
aws s3 sync publish/wwwroot/ \
    s3://my-blazor-wasm-app/ \
    --delete
```

The `--delete` parameter removes objects from Amazon S3 that no longer exist in the current build directory.

However, Blazor WebAssembly applications include several special file types, such as:

- `.wasm`
- `.wasm.br`
- `.js.br`
- `.dat.br`

If their metadata is not configured correctly, the browser may not be able to load or decompress these files. Therefore, different groups of files should be uploaded with the appropriate Content-Type and Content-Encoding metadata.

### Uploading Framework Files

Upload framework files that are not pre-compressed:

```bash
aws s3 sync publish/wwwroot/_framework/ \
    s3://my-blazor-wasm-app/_framework/ \
    --delete \
    --exclude "*.br" \
    --cache-control "max-age=31536000,immutable"
```

### Uploading Brotli-compressed WebAssembly Files

```bash
aws s3 sync publish/wwwroot/_framework/ \
    s3://my-blazor-wasm-app/_framework/ \
    --exclude "*" \
    --include "*.wasm.br" \
    --content-encoding "br" \
    --content-type "application/wasm" \
    --cache-control "max-age=31536000,immutable"
```

### Uploading Brotli-compressed JavaScript Files

```bash
aws s3 sync publish/wwwroot/_framework/ \
    s3://my-blazor-wasm-app/_framework/ \
    --exclude "*" \
    --include "*.js.br" \
    --content-encoding "br" \
    --content-type "application/javascript" \
    --cache-control "max-age=31536000,immutable"
```

### Uploading Brotli-compressed Data Files

```bash
aws s3 sync publish/wwwroot/_framework/ \
    s3://my-blazor-wasm-app/_framework/ \
    --exclude "*" \
    --include "*.dat.br" \
    --content-encoding "br" \
    --content-type "application/octet-stream" \
    --cache-control "max-age=31536000,immutable"
```

### Uploading Static Assets

```bash
aws s3 sync publish/wwwroot/ \
    s3://my-blazor-wasm-app/ \
    --exclude "_framework/*" \
    --exclude "index.html" \
    --cache-control "max-age=31536000,immutable"
```

### Uploading index.html

The `index.html` file should always be retrieved in its latest version:

```bash
aws s3 cp publish/wwwroot/index.html \
    s3://my-blazor-wasm-app/index.html \
    --content-type "text/html" \
    --cache-control "no-cache,no-store,must-revalidate"
```

Configuring the appropriate cache settings prevents an outdated `index.html` file from referring to framework files that no longer exist after a new version is deployed.

---

## Clearing the CloudFront Cache

After uploading a new version, create a CloudFront invalidation to remove old content from the cache:

```bash
aws cloudfront create-invalidation \
    --distribution-id EXAMPLE123456 \
    --paths "/*"
```

Where:

- `EXAMPLE123456` is the CloudFront distribution ID.
- `/*` clears the cache for every application path.

For larger systems, it may be more efficient to invalidate only the required files:

```bash
aws cloudfront create-invalidation \
    --distribution-id EXAMPLE123456 \
    --paths "/index.html"
```

Because files in `_framework` usually contain content hashes in their filenames, invalidating only `index.html` is generally sufficient.

---

## Automating the Deployment Process

To reduce the number of manually executed commands, create a `deploy.sh` script.

```bash
#!/bin/bash

set -e

BUCKET_NAME=$1
DISTRIBUTION_ID=$2

if [ -z "$BUCKET_NAME" ] || [ -z "$DISTRIBUTION_ID" ]; then
  echo "Usage: ./deploy.sh <bucket-name> <distribution-id>"
  exit 1
fi

echo "Building Blazor WebAssembly application..."

dotnet publish src/BlazorApp/BlazorApp.csproj \
  -c Release \
  -o publish \
  --nologo

echo "Uploading framework files..."

aws s3 sync publish/wwwroot/_framework/ \
  s3://$BUCKET_NAME/_framework/ \
  --delete \
  --exclude "*.br" \
  --cache-control "max-age=31536000,immutable"

echo "Uploading static assets..."

aws s3 sync publish/wwwroot/ \
  s3://$BUCKET_NAME/ \
  --exclude "_framework/*" \
  --exclude "index.html" \
  --cache-control "max-age=31536000,immutable"

echo "Uploading index.html..."

aws s3 cp publish/wwwroot/index.html \
  s3://$BUCKET_NAME/index.html \
  --content-type "text/html" \
  --cache-control "no-cache,no-store,must-revalidate"

echo "Creating CloudFront invalidation..."

aws cloudfront create-invalidation \
  --distribution-id $DISTRIBUTION_ID \
  --paths "/index.html"

echo "Deployment completed."
```

Grant execution permission to the script:

```bash
chmod +x scripts/deploy.sh
```

Run the script:

```bash
./scripts/deploy.sh \
    my-blazor-wasm-app \
    EXAMPLE123456
```

The script performs three main tasks:

1. Builds the Blazor WebAssembly application.
2. Uploads the files to Amazon S3.
3. Invalidates the CloudFront cache after deployment.

---

## Verifying the Deployment

After the upload is complete, open the `app_url` returned by Terraform:

```text
https://example.cloudfront.net
```

The Blazor WebAssembly application should load through CloudFront.

<div style="text-align: center;">
  <img src="/intership-report/images/Blog/3.2/3.png"
       alt="Blazor WebAssembly application running through CloudFront"
       width="900">
</div>

<p align="center">
  <i>Figure 3. Blazor WebAssembly application accessed through Amazon CloudFront.</i>
</p>

### Checking the Cache-Control Header of index.html

```bash
curl -sI \
    https://example.cloudfront.net/index.html
```

Expected result:

```text
cache-control: no-cache,no-store,must-revalidate
```

### Checking the Content-Type of a WebAssembly File

```bash
curl -sI \
    https://example.cloudfront.net/_framework/dotnet.runtime.wasm
```

Expected result:

```text
content-type: application/wasm
```

### Checking the Framework Cache Policy

```bash
curl -sI \
    https://example.cloudfront.net/_framework/blazor.web.js
```

Expected result:

```text
cache-control: max-age=31536000,immutable
```

### Testing Client-side Routing

Open the following path directly in a new browser tab:

```text
https://example.cloudfront.net/counter
```

If the Counter page is displayed correctly, the CloudFront Custom Error Response configuration is working properly.

---

## Using a Custom Domain

By default, CloudFront provides a domain in the following format:

```text
https://example.cloudfront.net
```

If a domain is managed using Amazon Route 53, a custom address can be configured:

```text
https://app.example.com
```

The following steps are required:

1. Request an SSL/TLS certificate from AWS Certificate Manager.
2. Create the certificate in the `us-east-1` Region because CloudFront requires certificates from this Region.
3. Verify domain ownership using DNS validation.
4. Add the custom domain to the CloudFront `aliases` property.
5. Configure CloudFront to use the ACM certificate.
6. Create an Amazon Route 53 Alias record pointing to the CloudFront distribution.

Example Terraform configuration:

```hcl
viewer_certificate {
  acm_certificate_arn      = aws_acm_certificate_validation.blazor.certificate_arn
  ssl_support_method       = "sni-only"
  minimum_protocol_version = "TLSv1.2_2021"
}
```

Define the domain alias:

```hcl
aliases = ["app.example.com"]
```

Create an Amazon Route 53 record:

```hcl
resource "aws_route53_record" "blazor_app" {
  zone_id = var.route53_zone_id
  name    = var.custom_domain
  type    = "A"

  alias {
    name                   = aws_cloudfront_distribution.blazor_app.domain_name
    zone_id                = aws_cloudfront_distribution.blazor_app.hosted_zone_id
    evaluate_target_health = false
  }
}
```

After updating the configuration, run:

```bash
terraform apply
```

---

## Common Errors

### Access Denied When Opening the Website

Possible causes include:

- The S3 bucket policy does not permit CloudFront access.
- `AWS:SourceArn` does not match the CloudFront distribution.
- Origin Access Control has not been attached to the origin.
- The `index.html` file has not been uploaded to Amazon S3.

Review the bucket policy and Origin Access Control configuration in CloudFront.

### A Child Route Returns 403 or 404

This usually occurs when CloudFront Custom Error Responses have not been configured.

Configure the following responses:

```text
403 → /index.html → HTTP 200
404 → /index.html → HTTP 200
```

### The Browser Cannot Load WASM Files

The most common cause is an incorrect Content-Type for `.wasm` files.

The correct value is:

```text
application/wasm
```

For `.wasm.br` files, configure:

```text
Content-Encoding: br
Content-Type: application/wasm
```

### The Website Still Displays an Old Version

The `index.html` file may still be cached by the browser or CloudFront.

Recommended actions:

- Set the `index.html` Cache-Control header to `no-cache`.
- Create a CloudFront invalidation for `/index.html`.
- Review the default cache policy of the CloudFront distribution.

### The S3 Bucket Name Already Exists

Amazon S3 bucket names must be globally unique.

Choose another name, for example:

```hcl
bucket_name = "linh-blazor-wasm-app-2026"
```

### CloudFront Is Not Immediately Available

After creating or updating a distribution, CloudFront requires time to distribute the configuration to edge locations.

Check the distribution status:

```bash
aws cloudfront get-distribution \
    --id EXAMPLE123456 \
    --query "Distribution.Status"
```

When the status changes to:

```text
Deployed
```

the distribution is ready to serve the application.

---

## Possible Extensions

The architecture can be extended with additional AWS services to meet practical requirements.

- Use AWS WAF to protect CloudFront from malicious requests.
- Configure a custom domain using Amazon Route 53.
- Manage HTTPS certificates using AWS Certificate Manager.
- Automate deployment with GitHub Actions or AWS CodePipeline.
- Store CloudFront access logs in Amazon S3.
- Monitor CloudFront using Amazon CloudWatch.
- Configure S3 Lifecycle rules to manage older object versions.
- Integrate Amazon API Gateway and AWS Lambda when the application requires backend functionality.
- Integrate Amazon Cognito for user authentication.
- Automatically run Terraform when infrastructure code changes.

A CI/CD process could follow this workflow:

```text
GitHub
   ↓
GitHub Actions
   ↓
dotnet publish
   ↓
Amazon S3
   ↓
CloudFront Invalidation
   ↓
Users
```

When source code is pushed to the main branch, the pipeline can automatically build the application, upload the new files to Amazon S3, and refresh the CloudFront cache.

---

## Advantages and Limitations

### Advantages

- No web server or .NET server management is required.
- The application can automatically scale with incoming traffic.
- CloudFront reduces latency for users in different geographical locations.
- The S3 bucket is protected using Origin Access Control.
- HTTPS is supported through CloudFront.
- The architecture is cost-effective for static applications.
- Infrastructure can be recreated consistently using Terraform.
- The solution is suitable for Single Page Applications.
- It can be integrated with a CI/CD pipeline.
- Custom domains and SSL/TLS certificates are supported.

### Limitations

- MIME types must be configured correctly for WebAssembly and Brotli files.
- Client-side routing requires additional CloudFront configuration.
- Terraform configuration may be difficult for beginners.
- CloudFront distribution updates are not applied immediately.
- Frequently invalidating the entire CloudFront cache may create additional costs.
- Blazor WebAssembly downloads the .NET runtime and libraries to the browser, so the first page load may be slower than a basic HTML website.
- Server-side functionality still requires Amazon API Gateway, AWS Lambda, or another backend service.

---

## Conclusion

Combining Amazon S3 and Amazon CloudFront provides an effective solution for hosting a .NET Blazor WebAssembly application. After compilation, the application consists entirely of static files, so it does not require a dedicated server or a .NET runtime environment on the backend.

Amazon S3 stores the application content, while CloudFront provides HTTPS, distributes files through a global network of edge locations, and improves access speed. Origin Access Control protects the S3 bucket by allowing only the specified CloudFront distribution to access its objects.

Terraform also makes it possible to manage the entire infrastructure as code, enabling consistent deployments and reducing errors caused by manual configuration. Applying separate cache policies to `index.html` and the files inside `_framework` provides a balance between application performance and reliable version updates.

This architecture is not limited to Blazor WebAssembly. It can also be applied to other Single Page Applications developed using React, Angular, Vue, and Svelte. When backend functionality is required, the system can be expanded using Amazon API Gateway, AWS Lambda, Amazon DynamoDB, and Amazon Cognito.

---

**Reference**: https://aws.amazon.com/vi/blogs/dotnet/host-a-net-blazor-webassembly-app-on-amazon-s3-and-amazon-cloudfront/

