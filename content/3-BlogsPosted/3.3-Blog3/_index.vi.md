---

title: "Blog 3"
date: 2026-07-20
weight: 3
chapter: false
pre: " <b> 3.3. </b> "

---

# TRIỂN KHAI ỨNG DỤNG .NET BLAZOR WEBASSEMBLY TRÊN AMAZON S3 VÀ AMAZON CLOUDFRONT

Blazor WebAssembly là framework của Microsoft cho phép xây dựng các ứng dụng web bằng ngôn ngữ C# và chạy trực tiếp trên trình duyệt thông qua WebAssembly. Sau khi biên dịch, ứng dụng tạo ra các tệp tĩnh như HTML, CSS, JavaScript và các thư viện .NET nên có thể triển khai trên các dịch vụ lưu trữ website tĩnh.

Bài viết trên AWS .NET Blog giới thiệu cách triển khai ứng dụng Blazor WebAssembly bằng Amazon S3 kết hợp Amazon CloudFront. Amazon S3 được sử dụng để lưu trữ toàn bộ mã nguồn đã publish, trong khi Amazon CloudFront đóng vai trò CDN giúp tăng tốc độ truy cập, giảm độ trễ và phân phối nội dung đến người dùng thông qua mạng lưới Edge Location của AWS. Ngoài ra, bài viết còn sử dụng Terraform để tự động hóa quá trình tạo hạ tầng trên AWS.

<div style="text-align: center;">
  <img src="/intership-report/images/Blog/3.3/3.png"
       alt="Kiến trúc triển khai Blazor WebAssembly"
       width="900">
</div>

<p align="center">
  <i>Hình 1. Kiến trúc triển khai Blazor WebAssembly trên Amazon S3 và Amazon CloudFront.</i>
</p>

### Các thành phần chính của hệ thống

**Blazor WebAssembly**

Blazor WebAssembly là ứng dụng phía client được xây dựng bằng .NET. Sau khi thực hiện lệnh `dotnet publish`, toàn bộ ứng dụng sẽ được biên dịch thành các tệp tĩnh để triển khai lên dịch vụ lưu trữ.

**Amazon S3**

Amazon S3 được sử dụng làm nơi lưu trữ toàn bộ website tĩnh sau khi publish. Dịch vụ này có khả năng mở rộng cao, độ bền dữ liệu lớn và chi phí thấp.

**Amazon CloudFront**

CloudFront hoạt động như một Content Delivery Network (CDN), giúp phân phối nội dung của website đến người dùng thông qua các Edge Location trên toàn thế giới, từ đó giảm thời gian tải trang và cải thiện hiệu năng.

**Terraform**

Terraform được sử dụng để tự động tạo các tài nguyên AWS như S3 Bucket, CloudFront Distribution và các cấu hình liên quan. Việc sử dụng Infrastructure as Code giúp quá trình triển khai nhanh hơn và dễ dàng tái sử dụng.

#### Lợi ích của kiến trúc

Việc kết hợp Amazon S3 với Amazon CloudFront mang lại nhiều lợi ích như:

- Giảm thời gian tải website nhờ mạng CDN toàn cầu.
- Tự động mở rộng khi số lượng người dùng tăng.
- Chi phí triển khai thấp vì không cần máy chủ.
- Dễ dàng triển khai và cập nhật phiên bản mới.
- Có thể tự động hóa toàn bộ hạ tầng bằng Terraform.

---

## Triển khai ứng dụng trên AWS

Sau khi hoàn thành ứng dụng Blazor WebAssembly, bước tiếp theo là publish mã nguồn và triển khai lên AWS.

### Bước 1: Publish ứng dụng

Sử dụng lệnh sau để tạo phiên bản Release của ứng dụng.

```bash
dotnet publish -c Release
```

Sau khi publish, toàn bộ tệp của website sẽ được lưu trong thư mục:

```text
bin/Release/net9.0/publish/wwwroot
```

Đây là thư mục sẽ được tải lên Amazon S3.

---

### Bước 2: Chuẩn bị Terraform

Clone mã nguồn Terraform được AWS cung cấp.

```bash
git clone https://github.com/aws-samples/aws-blazor-hosting.git

cd aws-blazor-hosting
```

Tiếp theo khởi tạo Terraform.

```bash
terraform init
```

---

### Bước 3: Triển khai hạ tầng

Sau khi cấu hình các biến cần thiết, triển khai toàn bộ hạ tầng bằng lệnh:

```bash
terraform apply
```

Terraform sẽ tạo các tài nguyên như:

- Amazon S3 Bucket
- Amazon CloudFront Distribution
- Origin Access Control (OAC)
- Bucket Policy
- Các cấu hình liên quan

Sau khi hoàn tất, hệ thống sẽ trả về địa chỉ CloudFront để truy cập website.

---

### Bước 4: Đồng bộ website lên Amazon S3

Đồng bộ toàn bộ nội dung website lên S3 bằng AWS CLI.

```bash
aws s3 sync ./publish/wwwroot s3://your-bucket-name
```

Sau khi upload hoàn tất, CloudFront sẽ phân phối website đến người dùng.

---

### Bước 5: Kiểm tra kết quả

Sau khi CloudFront hoàn tất quá trình triển khai (Deployment Completed), truy cập vào Domain Name của CloudFront để kiểm tra website.

Nếu cần cập nhật nội dung mới, chỉ cần publish lại ứng dụng và đồng bộ dữ liệu lên Amazon S3.

---

## Tăng tốc website với CloudFront

CloudFront lưu trữ bản sao của website tại nhiều Edge Location trên toàn cầu. Khi người dùng truy cập, dữ liệu sẽ được lấy từ vị trí gần nhất thay vì truy cập trực tiếp vào Amazon S3.

Nhờ đó:

- Giảm độ trễ.
- Tăng tốc độ tải trang.
- Giảm tải cho S3.
- Cải thiện trải nghiệm người dùng.

---

## Các hướng mở rộng

Sau khi triển khai thành công, hệ thống có thể mở rộng bằng nhiều dịch vụ khác của AWS như:

- Sử dụng AWS Certificate Manager (ACM) để cấu hình HTTPS.
- Kết nối Route 53 với tên miền riêng.
- Tích hợp AWS WAF nhằm tăng cường bảo mật.
- Tự động triển khai bằng GitHub Actions hoặc AWS CodePipeline.
- Theo dõi hoạt động bằng Amazon CloudWatch.

---

## Ưu điểm và hạn chế

### Ưu điểm

- Không cần quản lý máy chủ.
- Chi phí triển khai thấp.
- Website tải nhanh nhờ CloudFront.
- Tự động mở rộng theo lưu lượng truy cập.
- Có thể tự động hóa hạ tầng bằng Terraform.
- Dễ dàng cập nhật phiên bản mới.

### Hạn chế

- Chỉ phù hợp với website tĩnh hoặc ứng dụng phía client.
- Khi cập nhật nội dung cần thực hiện CloudFront Invalidation nếu muốn làm mới bộ nhớ đệm ngay lập tức.
- Cần cấu hình Bucket Policy và Origin Access Control chính xác để đảm bảo bảo mật.

---

## Kết luận

Việc triển khai Blazor WebAssembly trên Amazon S3 kết hợp Amazon CloudFront là một giải pháp hiệu quả để xây dựng các ứng dụng web hiện đại theo kiến trúc serverless. Amazon S3 đảm nhận vai trò lưu trữ website, trong khi CloudFront giúp tăng tốc độ truy cập thông qua mạng CDN toàn cầu. Bên cạnh đó, Terraform hỗ trợ tự động hóa toàn bộ quá trình tạo hạ tầng, giúp việc triển khai trở nên nhanh chóng, nhất quán và dễ dàng quản lý.

---

**Nguồn tham khảo**


https://aws.amazon.com/vi/blogs/dotnet/host-a-net-blazor-webassembly-app-on-amazon-s3-and-amazon-cloudfront/