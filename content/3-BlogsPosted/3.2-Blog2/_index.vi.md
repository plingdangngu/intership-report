
---
title: "Blog 2"
date: 2026-07-20
weight: 2
chapter: false
pre: " <b> 3.2. </b> "
---

# TRIỂN KHAI ỨNG DỤNG .NET BLAZOR WEBASSEMBLY TRÊN AMAZON S3 VÀ AMAZON CLOUDFRONT

Blazor WebAssembly là một framework thuộc hệ sinh thái .NET, cho phép xây dựng các ứng dụng web tương tác bằng ngôn ngữ C#. Khác với các ứng dụng web truyền thống cần máy chủ xử lý giao diện, Blazor WebAssembly có thể thực thi trực tiếp trong trình duyệt của người dùng thông qua công nghệ WebAssembly.

Sau khi ứng dụng được biên dịch bằng lệnh `dotnet publish`, kết quả đầu ra bao gồm các tệp tĩnh như HTML, CSS, JavaScript, WebAssembly và các thư viện cần thiết. Do không yêu cầu một máy chủ .NET hoạt động liên tục, ứng dụng Blazor WebAssembly có thể được triển khai dưới dạng website tĩnh trên Amazon S3.

Để cải thiện tốc độ truy cập, hỗ trợ HTTPS và phân phối nội dung đến người dùng ở nhiều khu vực, Amazon CloudFront được sử dụng làm Content Delivery Network. CloudFront tiếp nhận yêu cầu từ người dùng, lưu nội dung vào bộ nhớ đệm tại các edge location và lấy dữ liệu từ S3 khi cần thiết.

Bên cạnh đó, hạ tầng của hệ thống có thể được quản lý bằng Terraform. Việc sử dụng Infrastructure as Code giúp quá trình tạo tài nguyên, cập nhật cấu hình và triển khai lại hệ thống trở nên nhất quán, nhanh chóng và hạn chế các sai sót do cấu hình thủ công.

<div style="text-align: center;">
  <img src="/intership-report/images/Blog/3.2/2.png"
       alt="Kiến trúc triển khai Blazor WebAssembly trên Amazon S3 và Amazon CloudFront"
       width="900">
</div>

<p align="center">
  <i>Hình 1. Kiến trúc triển khai ứng dụng .NET Blazor WebAssembly bằng Amazon S3 và Amazon CloudFront.</i>
</p>

### Tổng quan kiến trúc

Kiến trúc triển khai ứng dụng Blazor WebAssembly gồm ba thành phần chính:

1. **Blazor WebAssembly**: Ứng dụng .NET được biên dịch thành các tệp tĩnh.
2. **Amazon S3**: Lưu trữ toàn bộ mã nguồn đã được biên dịch của ứng dụng.
3. **Amazon CloudFront**: Phân phối nội dung đến người dùng thông qua mạng lưới edge location toàn cầu.

Trong mô hình này, Amazon S3 bucket được thiết lập ở trạng thái riêng tư. Người dùng không truy cập trực tiếp vào S3 mà gửi yêu cầu thông qua CloudFront.

CloudFront sử dụng Origin Access Control để ký request bằng AWS Signature Version 4 trước khi truy cập dữ liệu trong S3. Chính sách của S3 bucket chỉ cho phép một CloudFront distribution cụ thể đọc các object bên trong bucket.

Cơ chế này giúp nội dung được bảo vệ tốt hơn so với việc mở quyền truy cập công khai cho S3 bucket.

### Lợi ích khi kết hợp Amazon S3 và Amazon CloudFront

Việc triển khai Blazor WebAssembly trên Amazon S3 và CloudFront mang lại nhiều lợi ích cho quá trình vận hành ứng dụng.

**Không cần quản lý máy chủ**

Blazor WebAssembly được biên dịch thành các tệp tĩnh nên không cần duy trì một máy chủ web hoặc máy chủ .NET riêng. Nhờ đó, không cần thực hiện các công việc như cập nhật hệ điều hành, cài đặt bản vá hoặc quản lý tài nguyên máy chủ.

**Khả năng mở rộng**

Amazon S3 có khả năng lưu trữ số lượng lớn object, trong khi CloudFront có thể xử lý lượng truy cập tăng cao thông qua hệ thống edge location. Người triển khai không cần cấu hình Auto Scaling hoặc dự đoán trước số lượng người dùng.

**Tốc độ truy cập**

CloudFront lưu nội dung tại các edge location gần người dùng. Khi một nội dung đã được lưu trong cache, CloudFront có thể phản hồi trực tiếp mà không cần gửi yêu cầu về S3, từ đó giảm độ trễ khi tải ứng dụng.

**Hỗ trợ HTTPS**

CloudFront cung cấp địa chỉ truy cập HTTPS mặc định. Khi sử dụng tên miền riêng, có thể kết hợp CloudFront với AWS Certificate Manager để tạo và quản lý chứng chỉ SSL/TLS.

**Tối ưu chi phí**

Vì không có máy chủ chạy liên tục nên không phát sinh chi phí cho tài nguyên compute. Chi phí chủ yếu đến từ dung lượng lưu trữ trên S3, số lượng request và lượng dữ liệu được CloudFront truyền đến người dùng.

### Các quyết định thiết kế quan trọng

#### Sử dụng Origin Access Control

Origin Access Control, viết tắt là OAC, là cơ chế được CloudFront sử dụng để truy cập an toàn vào S3 origin.

Thay vì công khai S3 bucket, CloudFront sẽ ký từng yêu cầu bằng AWS Signature Version 4. Bucket policy chỉ cho phép CloudFront distribution đã được chỉ định truy cập các object.

Các lợi ích chính của OAC gồm:

- Không cần mở quyền truy cập công khai cho S3.
- Chỉ CloudFront distribution được chỉ định mới có thể đọc dữ liệu.
- Request từ CloudFront đến S3 được ký bằng AWS Signature Version 4.
- Giảm nguy cơ người dùng bỏ qua CloudFront và truy cập trực tiếp vào S3.
- Phù hợp với kiến trúc phân phối nội dung an toàn hiện nay.

#### Không bật S3 Static Website Hosting

Trong kiến trúc này, Amazon S3 chỉ được sử dụng để lưu trữ object. Tính năng Static Website Hosting của S3 không cần được bật vì CloudFront là điểm truy cập duy nhất từ Internet.

Bucket được cấu hình chặn toàn bộ quyền truy cập công khai. CloudFront kết nối đến regional endpoint của S3 thông qua Origin Access Control.

Cấu hình này giúp:

- Hạn chế việc truy cập trực tiếp vào S3.
- Bắt buộc người dùng truy cập thông qua HTTPS của CloudFront.
- Áp dụng thống nhất các quy tắc cache.
- Dễ dàng cấu hình tên miền và chứng chỉ bảo mật.

#### Phân tách chính sách cache

Không phải tất cả tệp của ứng dụng Blazor WebAssembly đều nên sử dụng cùng một thời gian cache.

Các tệp trong thư mục `_framework` thường có tên chứa mã hash được tạo trong quá trình build. Khi nội dung thay đổi, tên tệp cũng thay đổi. Vì vậy, những tệp này có thể được lưu cache trong thời gian dài.

Ngược lại, tệp `index.html` tham chiếu đến các tệp của phiên bản ứng dụng hiện tại. Nếu tệp này bị cache quá lâu, người dùng có thể tiếp tục tải phiên bản cũ sau khi hệ thống được cập nhật.

Vì vậy, kiến trúc sử dụng hai chiến lược cache:

- `_framework/*`: Cache trong một năm và đặt thuộc tính `immutable`.
- `index.html`: Không cache để luôn lấy phiên bản mới nhất.
- Các tệp CSS, hình ảnh và thư viện: Có thể cache dài hạn nếu tên tệp được quản lý theo phiên bản.

### Xử lý Client-side Routing

Blazor WebAssembly là một Single Page Application. Việc chuyển đổi giữa các trang được thực hiện ở phía trình duyệt thông qua Blazor Router.

Ví dụ, khi người dùng truy cập:

```text
https://example.com/counter
````

Trình duyệt sẽ gửi yêu cầu đến đường dẫn `/counter`. Tuy nhiên, trong Amazon S3 không tồn tại object nào có tên `counter`.

Do S3 bucket ở trạng thái riêng tư, yêu cầu này có thể nhận phản hồi `403 Forbidden`. Trong một số cấu hình khác, hệ thống có thể trả về `404 Not Found`.

Để xử lý vấn đề này, CloudFront được cấu hình Custom Error Response:

* Chuyển lỗi `403` sang tệp `/index.html`.
* Chuyển lỗi `404` sang tệp `/index.html`.
* Thay đổi mã trạng thái phản hồi thành `200`.

Sau khi `index.html` được tải, Blazor Router đọc URL hiện tại và hiển thị component tương ứng.

Cách xử lý này không chỉ áp dụng cho Blazor WebAssembly mà còn có thể được sử dụng với các framework Single Page Application như React, Angular, Vue và Svelte.

---

## Chuẩn bị môi trường

Trước khi bắt đầu triển khai, cần cài đặt và cấu hình các công cụ sau:

* .NET SDK phiên bản 10.0 trở lên.
* Terraform phiên bản 1.6 trở lên.
* AWS Command Line Interface phiên bản 2.
* Git để quản lý và tải mã nguồn.
* Một tài khoản AWS có quyền tạo Amazon S3 bucket và CloudFront distribution.

Kiểm tra phiên bản .NET SDK:

```bash
dotnet --version
```

Kiểm tra Terraform:

```bash
terraform --version
```

Kiểm tra AWS CLI:

```bash
aws --version
```

Cấu hình AWS CLI bằng Access Key hoặc thông tin xác thực phù hợp:

```bash
aws configure
```

Sau đó nhập các thông tin:

```text
AWS Access Key ID
AWS Secret Access Key
Default region name
Default output format
```

IAM user hoặc IAM role được sử dụng để triển khai cần có quyền quản lý các tài nguyên liên quan như:

* Amazon S3 bucket.
* S3 bucket policy.
* S3 Public Access Block.
* Amazon CloudFront distribution.
* CloudFront Origin Access Control.
* CloudFront cache policy.
* CloudFront invalidation.
* AWS Certificate Manager và Route 53 nếu sử dụng tên miền riêng.

---

## Tạo ứng dụng Blazor WebAssembly

### Bước 1: Khởi tạo dự án

Sử dụng lệnh sau để tạo một dự án Blazor WebAssembly mới với .NET 10:

```bash
dotnet new blazorwasm \
    -o src/BlazorApp \
    --framework net10.0
```

Cấu trúc thư mục cơ bản sau khi tạo dự án:

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

Template mặc định cung cấp một số trang mẫu như:

* Home.
* Counter.
* Weather.

Các trang này có thể được sử dụng để kiểm tra khả năng điều hướng, cập nhật giao diện và tải dữ liệu trong ứng dụng.

### Bước 2: Chạy thử ứng dụng

Trước khi triển khai lên AWS, cần kiểm tra ứng dụng trên môi trường local:

```bash
dotnet run --project src/BlazorApp
```

Sau khi ứng dụng khởi động, terminal sẽ hiển thị địa chỉ local tương tự:

```text
https://localhost:5001
```

Truy cập địa chỉ này trên trình duyệt và kiểm tra các trang Home, Counter và Weather.

### Bước 3: Build ứng dụng

Để tạo các tệp tĩnh dùng cho quá trình triển khai, sử dụng lệnh:

```bash
dotnet publish src/BlazorApp/BlazorApp.csproj \
    -c Release \
    -o publish \
    --nologo
```

Sau khi build hoàn tất, nội dung cần triển khai nằm trong thư mục:

```text
publish/wwwroot/
```

Thư mục này chứa các tệp như:

```text
publish/wwwroot/
├── _framework/
├── css/
├── sample-data/
├── index.html
├── favicon.png
└── icon-512.png
```

Toàn bộ nội dung bên trong `wwwroot` có thể được lưu trữ dưới dạng object trong Amazon S3.

---

## Xây dựng hạ tầng bằng Terraform

Terraform được sử dụng để tự động tạo Amazon S3 bucket, Origin Access Control, cache policy và CloudFront distribution.

Cấu trúc thư mục Terraform có thể được tổ chức như sau:

```text
infra/
├── main.tf
├── cloudfront.tf
├── variables.tf
├── outputs.tf
└── terraform.tfvars
```

Trong đó:

* `main.tf`: Khai báo S3 bucket, versioning và bucket policy.
* `cloudfront.tf`: Khai báo CloudFront, OAC và cache policy.
* `variables.tf`: Khai báo các biến đầu vào.
* `outputs.tf`: Xuất các thông tin sau khi triển khai.
* `terraform.tfvars`: Cung cấp giá trị thực tế cho các biến.

### Bước 1: Khai báo AWS Provider

Trong file `main.tf`, cấu hình AWS Provider:

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

### Bước 2: Tạo Amazon S3 Bucket

Thêm cấu hình tạo S3 bucket:

```hcl
resource "aws_s3_bucket" "blazor_app" {
  bucket = var.bucket_name

  tags = {
    Environment = var.environment
    ManagedBy   = "terraform"
  }
}
```

Tên S3 bucket phải là duy nhất trên phạm vi toàn cầu. Vì vậy, cần thay giá trị `bucket_name` bằng một tên riêng.

### Bước 3: Chặn truy cập công khai

S3 bucket chỉ được truy cập thông qua CloudFront nên cần bật Public Access Block:

```hcl
resource "aws_s3_bucket_public_access_block" "blazor_app" {
  bucket = aws_s3_bucket.blazor_app.id

  block_public_acls       = true
  block_public_policy     = true
  ignore_public_acls      = true
  restrict_public_buckets = true
}
```

Cấu hình này ngăn việc thêm ACL hoặc bucket policy có thể làm cho dữ liệu trong S3 bị công khai ngoài ý muốn.

### Bước 4: Bật Versioning

S3 Versioning giúp lưu giữ nhiều phiên bản của cùng một object:

```hcl
resource "aws_s3_bucket_versioning" "blazor_app" {
  bucket = aws_s3_bucket.blazor_app.id

  versioning_configuration {
    status = "Enabled"
  }
}
```

Nếu một tệp bị ghi đè hoặc xóa nhầm, phiên bản cũ vẫn có thể được khôi phục.

### Bước 5: Tạo Origin Access Control

Trong file `cloudfront.tf`, khai báo Origin Access Control:

```hcl
resource "aws_cloudfront_origin_access_control" "blazor_app" {
  name                              = "${var.bucket_name}-oac"
  description                       = "OAC for Blazor WebAssembly application"
  origin_access_control_origin_type = "s3"
  signing_behavior                  = "always"
  signing_protocol                  = "sigv4"
}
```

Thuộc tính `signing_behavior = "always"` yêu cầu CloudFront ký mọi request gửi đến S3 bằng AWS Signature Version 4.

### Bước 6: Tạo chính sách cache dài hạn

Tạo cache policy cho các tệp có tên chứa mã hash:

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

Giá trị `31536000` tương ứng với một năm tính theo giây.

### Bước 7: Tạo chính sách không cache

Tạo cache policy dành cho `index.html` và các đường dẫn không khớp với thư mục `_framework`:

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

Chính sách này đảm bảo CloudFront luôn lấy phiên bản mới nhất của `index.html`.

### Bước 8: Tạo CloudFront Distribution

Khai báo CloudFront distribution:

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

Một số cấu hình quan trọng:

* `default_root_object = "index.html"`: Tải trang chính khi truy cập domain.
* `redirect-to-https`: Tự động chuyển yêu cầu HTTP sang HTTPS.
* `_framework/*`: Áp dụng cache dài hạn cho các tệp Blazor.
* `403` và `404`: Chuyển về `index.html` để hỗ trợ client-side routing.
* `compress = true`: Cho phép CloudFront nén nội dung khi phân phối.

### Bước 9: Cấu hình S3 Bucket Policy

Bucket policy chỉ cho phép CloudFront distribution vừa tạo đọc object:

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

Điều kiện `AWS:SourceArn` giới hạn quyền truy cập cho đúng CloudFront distribution của hệ thống.

### Bước 10: Khai báo biến

Trong file `variables.tf`:

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

### Bước 11: Cung cấp giá trị cho biến

Tạo file `terraform.tfvars`:

```hcl
aws_region             = "us-west-2"
bucket_name            = "my-blazor-wasm-app"
environment            = "prod"
cloudfront_price_class = "PriceClass_100"
```

Cần thay `my-blazor-wasm-app` bằng một tên S3 bucket chưa được sử dụng.

### Bước 12: Khai báo output

Trong file `outputs.tf`:

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

Các output này được sử dụng trong quá trình upload ứng dụng và xóa cache CloudFront.

---

## Triển khai hạ tầng

Di chuyển vào thư mục Terraform:

```bash
cd infra
```

Khởi tạo Terraform:

```bash
terraform init
```

Kiểm tra các tài nguyên sẽ được tạo:

```bash
terraform plan
```

Triển khai hạ tầng:

```bash
terraform apply
```

Nhập:

```text
yes
```

Sau khi quá trình triển khai hoàn tất, Terraform hiển thị kết quả tương tự:

```text
Outputs:

s3_bucket_name               = "my-blazor-wasm-app"
cloudfront_distribution_id   = "EXAMPLE123456"
app_url                      = "https://example.cloudfront.net"
```

CloudFront cần một khoảng thời gian để phân phối cấu hình đến các edge location trước khi có thể sử dụng.

<div style="text-align: center;">
  <img src="/intership-report/images/Blog/3.2/2.png"
       alt="Kết quả triển khai hạ tầng bằng Terraform"
       width="900">
</div>

<p align="center">
  <i>Hình 2. Kết quả tạo Amazon S3 bucket và CloudFront distribution bằng Terraform.</i>
</p>

---

## Upload ứng dụng lên Amazon S3

Sau khi hạ tầng được tạo, cần build và upload các tệp trong thư mục `publish/wwwroot` lên S3.

### Upload các tệp thông thường

Có thể sử dụng lệnh sau:

```bash
aws s3 sync publish/wwwroot/ \
    s3://my-blazor-wasm-app/ \
    --delete
```

Tham số `--delete` xóa các object trong S3 không còn tồn tại trong thư mục build hiện tại.

Tuy nhiên, ứng dụng Blazor WebAssembly có nhiều loại tệp đặc biệt như:

* `.wasm`
* `.wasm.br`
* `.js.br`
* `.dat.br`

Nếu metadata không được cấu hình đúng, trình duyệt có thể không tải hoặc giải nén được các tệp này. Vì vậy, cần upload từng nhóm tệp với Content-Type và Content-Encoding phù hợp.

### Upload tệp framework

Upload các tệp chưa được nén sẵn:

```bash
aws s3 sync publish/wwwroot/_framework/ \
    s3://my-blazor-wasm-app/_framework/ \
    --delete \
    --exclude "*.br" \
    --cache-control "max-age=31536000,immutable"
```

### Upload tệp WebAssembly nén bằng Brotli

```bash
aws s3 sync publish/wwwroot/_framework/ \
    s3://my-blazor-wasm-app/_framework/ \
    --exclude "*" \
    --include "*.wasm.br" \
    --content-encoding "br" \
    --content-type "application/wasm" \
    --cache-control "max-age=31536000,immutable"
```

### Upload tệp JavaScript nén bằng Brotli

```bash
aws s3 sync publish/wwwroot/_framework/ \
    s3://my-blazor-wasm-app/_framework/ \
    --exclude "*" \
    --include "*.js.br" \
    --content-encoding "br" \
    --content-type "application/javascript" \
    --cache-control "max-age=31536000,immutable"
```

### Upload tệp dữ liệu nén bằng Brotli

```bash
aws s3 sync publish/wwwroot/_framework/ \
    s3://my-blazor-wasm-app/_framework/ \
    --exclude "*" \
    --include "*.dat.br" \
    --content-encoding "br" \
    --content-type "application/octet-stream" \
    --cache-control "max-age=31536000,immutable"
```

### Upload các static asset

```bash
aws s3 sync publish/wwwroot/ \
    s3://my-blazor-wasm-app/ \
    --exclude "_framework/*" \
    --exclude "index.html" \
    --cache-control "max-age=31536000,immutable"
```

### Upload index.html

Tệp `index.html` phải luôn được lấy ở phiên bản mới nhất:

```bash
aws s3 cp publish/wwwroot/index.html \
    s3://my-blazor-wasm-app/index.html \
    --content-type "text/html" \
    --cache-control "no-cache,no-store,must-revalidate"
```

Việc cấu hình cache phù hợp giúp tránh tình trạng `index.html` cũ tham chiếu đến các tệp framework không còn tồn tại sau khi triển khai phiên bản mới.

---

## Xóa cache CloudFront

Sau khi upload phiên bản mới, có thể tạo một CloudFront invalidation để xóa nội dung cũ khỏi cache:

```bash
aws cloudfront create-invalidation \
    --distribution-id EXAMPLE123456 \
    --paths "/*"
```

Trong đó:

* `EXAMPLE123456` là CloudFront distribution ID.
* `/*` yêu cầu xóa cache của tất cả đường dẫn.

Đối với hệ thống lớn, có thể chỉ invalidation những tệp cần thiết như:

```bash
aws cloudfront create-invalidation \
    --distribution-id EXAMPLE123456 \
    --paths "/index.html"
```

Do các tệp trong `_framework` có tên chứa mã hash, thông thường chỉ cần xóa cache của `index.html`.

---

## Tự động hóa quá trình triển khai

Để hạn chế việc thực hiện nhiều lệnh thủ công, có thể tạo script `deploy.sh`.

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

Cấp quyền thực thi cho script:

```bash
chmod +x scripts/deploy.sh
```

Chạy script:

```bash
./scripts/deploy.sh \
    my-blazor-wasm-app \
    EXAMPLE123456
```

Script sẽ thực hiện ba công việc chính:

1. Build ứng dụng Blazor WebAssembly.
2. Upload các tệp lên Amazon S3.
3. Xóa cache của CloudFront sau khi triển khai.

---

## Kiểm tra kết quả triển khai

Sau khi upload hoàn tất, truy cập địa chỉ `app_url` được Terraform trả về:

```text
https://example.cloudfront.net
```

Ứng dụng Blazor WebAssembly sẽ được tải thông qua CloudFront.

<div style="text-align: center;">
  <img src="/intership-report/images/Blog/3.2/3.png"
       alt="Ứng dụng Blazor WebAssembly hoạt động trên CloudFront"
       width="900">
</div>

<p align="center">
  <i>Hình 3. Ứng dụng Blazor WebAssembly được truy cập thông qua Amazon CloudFront.</i>
</p>

### Kiểm tra Cache-Control của index.html

```bash
curl -sI \
    https://example.cloudfront.net/index.html
```

Kết quả mong đợi:

```text
cache-control: no-cache,no-store,must-revalidate
```

### Kiểm tra Content-Type của tệp WebAssembly

```bash
curl -sI \
    https://example.cloudfront.net/_framework/dotnet.runtime.wasm
```

Kết quả mong đợi:

```text
content-type: application/wasm
```

### Kiểm tra cache của framework

```bash
curl -sI \
    https://example.cloudfront.net/_framework/blazor.web.js
```

Kết quả mong đợi:

```text
cache-control: max-age=31536000,immutable
```

### Kiểm tra client-side routing

Mở trực tiếp đường dẫn sau trong một tab trình duyệt mới:

```text
https://example.cloudfront.net/counter
```

Nếu trang Counter được hiển thị bình thường, cấu hình Custom Error Response của CloudFront đã hoạt động đúng.

---

## Sử dụng tên miền riêng

Theo mặc định, CloudFront cung cấp một domain có dạng:

```text
https://example.cloudfront.net
```

Nếu có tên miền được quản lý bằng Amazon Route 53, có thể cấu hình domain riêng như:

```text
https://app.example.com
```

Khi triển khai tên miền riêng, cần thực hiện các bước:

1. Tạo chứng chỉ SSL/TLS trong AWS Certificate Manager.
2. Chứng chỉ phải được tạo tại Region `us-east-1` để sử dụng với CloudFront.
3. Xác thực quyền sở hữu domain bằng DNS.
4. Thêm domain vào thuộc tính `aliases` của CloudFront.
5. Cấu hình CloudFront sử dụng ACM certificate.
6. Tạo Route 53 Alias Record trỏ đến CloudFront distribution.

Ví dụ cấu hình Terraform:

```hcl
viewer_certificate {
  acm_certificate_arn      = aws_acm_certificate_validation.blazor.certificate_arn
  ssl_support_method       = "sni-only"
  minimum_protocol_version = "TLSv1.2_2021"
}
```

Khai báo alias:

```hcl
aliases = ["app.example.com"]
```

Tạo bản ghi Route 53:

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

Sau khi cập nhật cấu hình, chạy lại:

```bash
terraform apply
```

---

## Các lỗi thường gặp

### Lỗi Access Denied khi truy cập website

Nguyên nhân có thể đến từ:

* S3 bucket policy chưa cho phép CloudFront truy cập.
* `AWS:SourceArn` không đúng với CloudFront distribution.
* Origin Access Control chưa được gắn vào origin.
* Tệp `index.html` chưa được upload lên S3.

Cần kiểm tra bucket policy và Origin Access Control trong CloudFront.

### Truy cập trang con trả về 403 hoặc 404

Nguyên nhân là CloudFront chưa được cấu hình Custom Error Response.

Cần cấu hình:

```text
403 → /index.html → HTTP 200
404 → /index.html → HTTP 200
```

### Trình duyệt không tải được tệp WASM

Nguyên nhân thường do Content-Type của tệp `.wasm` không chính xác.

Content-Type đúng là:

```text
application/wasm
```

Đối với tệp `.wasm.br`, cần thêm:

```text
Content-Encoding: br
Content-Type: application/wasm
```

### Website vẫn hiển thị phiên bản cũ

Nguyên nhân có thể là `index.html` đang được lưu trong cache của trình duyệt hoặc CloudFront.

Cách xử lý:

* Đặt Cache-Control của `index.html` thành `no-cache`.
* Tạo CloudFront invalidation cho `/index.html`.
* Kiểm tra lại cache policy mặc định của distribution.

### S3 Bucket Name đã tồn tại

Tên S3 bucket phải duy nhất trên toàn bộ AWS.

Cần đổi sang tên khác, chẳng hạn:

```hcl
bucket_name = "linh-blazor-wasm-app-2026"
```

### CloudFront chưa hoạt động ngay

Sau khi tạo hoặc cập nhật distribution, CloudFront cần thời gian để phân phối cấu hình đến các edge location.

Có thể kiểm tra trạng thái distribution:

```bash
aws cloudfront get-distribution \
    --id EXAMPLE123456 \
    --query "Distribution.Status"
```

Khi trạng thái chuyển sang:

```text
Deployed
```

distribution đã sẵn sàng hoạt động.

---

## Các hướng mở rộng

Kiến trúc có thể được mở rộng thêm nhiều thành phần để đáp ứng nhu cầu thực tế.

* Sử dụng AWS WAF để bảo vệ CloudFront trước các request độc hại.
* Sử dụng tên miền riêng thông qua Amazon Route 53.
* Quản lý chứng chỉ HTTPS bằng AWS Certificate Manager.
* Tự động triển khai bằng GitHub Actions hoặc AWS CodePipeline.
* Lưu access log của CloudFront vào Amazon S3.
* Theo dõi hoạt động của CloudFront bằng Amazon CloudWatch.
* Thiết lập S3 Lifecycle để quản lý các phiên bản object cũ.
* Tích hợp API Gateway và AWS Lambda nếu ứng dụng cần backend.
* Tích hợp Amazon Cognito để xác thực người dùng.
* Tự động chạy Terraform khi có thay đổi hạ tầng.

Có thể xây dựng quy trình CI/CD theo luồng:

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
Người dùng
```

Khi mã nguồn được cập nhật lên nhánh chính, pipeline sẽ tự động build ứng dụng, upload tệp mới lên S3 và làm mới cache của CloudFront.

---

## Ưu điểm và hạn chế

### Ưu điểm

* Không cần quản lý máy chủ web hoặc máy chủ .NET.
* Có khả năng tự động mở rộng theo lưu lượng truy cập.
* CloudFront giúp giảm độ trễ khi người dùng truy cập từ nhiều khu vực.
* S3 bucket được bảo vệ bằng Origin Access Control.
* Hỗ trợ HTTPS thông qua CloudFront.
* Chi phí thấp đối với ứng dụng tĩnh.
* Dễ dàng tái tạo hạ tầng bằng Terraform.
* Phù hợp với các ứng dụng Single Page Application.
* Có thể tích hợp với quy trình CI/CD.
* Hỗ trợ tên miền riêng và chứng chỉ SSL/TLS.

### Hạn chế

* Cần cấu hình đúng MIME type cho các tệp WebAssembly và Brotli.
* Cần xử lý riêng client-side routing trên CloudFront.
* Việc cấu hình Terraform có thể khó đối với người mới.
* CloudFront cần một khoảng thời gian để cập nhật distribution.
* Việc invalidation toàn bộ cache thường xuyên có thể phát sinh thêm chi phí.
* Blazor WebAssembly tải .NET runtime và thư viện xuống trình duyệt nên lần tải đầu tiên có thể chậm hơn website HTML thông thường.
* Các chức năng xử lý phía máy chủ vẫn cần thêm API Gateway, Lambda hoặc một backend riêng.

---

## Kết luận

Việc kết hợp Amazon S3 và Amazon CloudFront là một giải pháp phù hợp để triển khai ứng dụng .NET Blazor WebAssembly. Sau khi được biên dịch, ứng dụng chỉ bao gồm các tệp tĩnh nên không cần duy trì máy chủ hoặc môi trường chạy .NET ở phía backend.

Amazon S3 chịu trách nhiệm lưu trữ nội dung, trong khi CloudFront cung cấp HTTPS, phân phối nội dung qua mạng lưới edge location và cải thiện tốc độ truy cập. Origin Access Control giúp bảo vệ S3 bucket bằng cách chỉ cho phép CloudFront distribution được chỉ định truy cập dữ liệu.

Bên cạnh đó, Terraform hỗ trợ quản lý toàn bộ hạ tầng bằng mã nguồn, giúp quá trình triển khai có thể lặp lại và hạn chế sai sót khi cấu hình thủ công. Việc thiết lập chính sách cache riêng cho `index.html` và các tệp trong `_framework` cũng giúp cân bằng giữa hiệu suất và khả năng cập nhật phiên bản mới.

Mô hình này không chỉ phù hợp với Blazor WebAssembly mà còn có thể được áp dụng cho nhiều Single Page Application khác như React, Angular, Vue và Svelte. Khi cần bổ sung chức năng phía backend, hệ thống có thể tiếp tục được mở rộng bằng Amazon API Gateway, AWS Lambda, Amazon DynamoDB và Amazon Cognito.

---

**Nguồn tham khảo**: https://aws.amazon.com/vi/blogs/dotnet/host-a-net-blazor-webassembly-app-on-amazon-s3-and-amazon-cloudfront/


