---
title: "Nhật ký tuần 7"
date: 2026-07-20
weight: 1
chapter: false
pre: " <b> 1.7. </b> "
---



### Mục tiêu tuần 7

* Triển khai website tĩnh trên Amazon S3.
* Tìm hiểu dịch vụ Amazon CloudFront.
* Cấu hình hệ thống để website có thể truy cập thông qua Internet.
* chuẩn bị dự án 
### Các công việc thực hiện trong tuần

| Thứ | Công việc | Ngày bắt đầu | Ngày hoàn thành | Nguồn tài liệu |
| --- | --------- | ------------ | --------------- | -------------- |
| 2 | - Tìm hiểu tính năng Static Website Hosting của Amazon S3 <br> - Chuẩn bị mã nguồn website để triển khai | 22/06/2026 | 22/06/2026 | https://docs.aws.amazon.com/AmazonS3/latest/userguide/WebsiteHosting.html |
| 3 | - Tạo S3 Bucket phục vụ lưu trữ website <br> - Upload mã nguồn website lên Amazon S3 <br> - Cấu hình Static Website Hosting | 23/06/2026 | 23/06/2026 | https://docs.aws.amazon.com/AmazonS3/latest/userguide/WebsiteHosting.html |
| 4 | - Thiết lập Bucket Policy và quyền Public Read <br> - Kiểm tra khả năng truy cập website thông qua S3 Endpoint | 24/06/2026 | 24/06/2026 | https://docs.aws.amazon.com/AmazonS3/latest/userguide/access-policy-language-overview.html |
| 5 | - Tìm hiểu Amazon CloudFront <br> - Tạo CloudFront Distribution kết nối với Amazon S3 <br> - Kiểm tra tốc độ truy cập website | 25/06/2026 | 26/06/2026 | https://docs.aws.amazon.com/AmazonCloudFront/latest/DeveloperGuide/Introduction.html |
| 6 |  - Kiểm tra cơ chế Cache của CloudFront <br> - Hoàn thiện môi trường triển khai | 27/06/2026 | 27/06/2026 | https://docs.aws.amazon.com/AmazonCloudFront/latest/DeveloperGuide/ |

### Kết quả đạt được tuần 7

* Triển khai thành công website tĩnh trên Amazon S3.

* Hiểu được cơ chế hoạt động của Static Website Hosting trên AWS.

* Cấu hình thành công Bucket Policy để website có thể truy cập từ Internet.

* Tạo và cấu hình Amazon CloudFront Distribution nhằm tăng tốc độ truy cập website.

* Hiểu được vai trò của CDN trong việc giảm độ trễ và cải thiện hiệu suất tải nội dung.

