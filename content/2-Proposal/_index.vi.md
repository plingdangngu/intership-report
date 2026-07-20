---
title: "Bản đề xuất"
date: 2026-07-20
weight: 2
chapter: false
pre: " <b> 2. </b> "
---



## SmartStudy AI - Trợ lý học tập serverless dựa trên tài liệu  

### 1. Tóm tắt điều hành  
SmartStudy AI là nền tảng học tập trên web giúp sinh viên học từ chính tài liệu PDF của mình. Người dùng có thể tạo tài khoản, tải lên và quản lý tài liệu, đặt câu hỏi dựa trên nội dung tài liệu, tạo bài luyện tập, nộp đáp án và xem điểm cùng lời giải thích.

Ứng dụng sử dụng kiến trúc serverless trên AWS cho việc lưu trữ web, xác thực, API, lưu trữ tài liệu, xử lý bất đồng bộ, dữ liệu ứng dụng và giám sát. Do tài khoản dự án không thể sử dụng Amazon Bedrock, tác vụ AI được chuyển sang model Qwen 2.5 7B chạy bằng Ollama trên máy chủ AI local tự quản lý. Repository của dự án có thành phần Cloudflare relay để hỗ trợ kết nối tới dịch vụ AI local; chi tiết triển khai sẽ được xác minh khi hoàn thiện tài liệu kỹ thuật

### 2. Vấn đề cần giải quyết  
Sinh viên thường phải học từ nhiều tài liệu dài và rời rạc. Việc đọc, tóm tắt, tự tạo câu hỏi ôn tập và theo dõi kết quả theo cách thủ công tốn nhiều thời gian. Chatbot thông thường cũng có thể trả lời ngoài nội dung tài liệu, khiến người học khó kiểm chứng câu trả lời có thực sự dựa trên nguồn đã cung cấp hay không.

SmartStudy giải quyết các vấn đề trên bằng một không gian học tập thống nhất cho phép:

* Tải lên và tổ chức tài liệu PDF.
* Đặt câu hỏi dựa trên tài liệu đã tải lên.
* Nhận câu trả lời kèm tham chiếu tới nội dung liên quan.
* Tự động tạo bài luyện tập.
* Lưu hội thoại, lượt làm bài, điểm số và lời giải thích.

### 3. Giải pháp đề xuất  
SmartStudy cung cấp các luồng chính sau:

1. Người dùng đăng ký hoặc đăng nhập thông qua Amazon Cognito.
2.  Ứng dụng web gửi yêu cầu backend qua Amazon API Gateway.
3. Tài liệu được lưu trên Amazon S3 và đưa vào Amazon SQS để xử lý bất đồng bộ.
4. AWS Lambda xử lý tài liệu và lưu metadata, chunk, hội thoại, quiz cùng lượt làm bài trong Amazon DynamoDB.
5. Yêu cầu AI được chuyển đến model Ollama local thông qua cơ chế relay của dự án.
6. Amazon CloudWatch thu thập log, metric và alarm vận hành.



### 4. Kiến trúc giải pháp sơ bộ

#### Kiến trúc đề xuất ban đầu

Sơ đồ dưới đây ghi lại kiến trúc được đề xuất trước khi triển khai. Sơ đồ được giữ lại để thể hiện định hướng thiết kế ban đầu; một số dịch vụ trong sơ đồ sau đó đã bị loại bỏ hoặc thay thế do giới hạn dịch vụ và yêu cầu vận hành thực tế.

<div style="text-align: center;">
  <img src="/intership-report/images/Gallery/OLD_Diagram.jpg"
       alt="Preliminary SmartStudy AI Architecture"
       width="900">
</div>

<p align="center">
  <i>Hình 1. Kiến trúc đề xuất ban đầu của hệ thống SmartStudy AI.</i>
</p>

#### Kiến trúc sau điều chỉnh

<div style="text-align: center;">
  <img src="/intership-report/images/Gallery/NEW_Diagram.jpg"
       alt="Final SmartStudy AI Architecture"
       width="900">
</div>

<p align="center">
  <i>Hình 2. Kiến trúc triển khai cuối cùng, gồm workload serverless trên AWS và môi trường AI Ollama tự quản lý.</i>
</p>

Dự án duy trì tài nguyên staging và production riêng biệt tại region (`*.us-east-1`). Branch staging trên GitHub được dùng cho quá trình tích hợp; các thay đổi đã được kiểm tra được merge vào branch main và triển khai production thông qua AWS Amplify.


### 5. Dịch vụ và công nghệ sử dụng
#### Dịch vụ AWS 
- **AWS Amplify Hosting**: Build và triển khai frontend từ GitHub.
- **Amazon Cognito**: Quản lý đăng ký, đăng nhập và token người dùng.
- **Amazon API Gateway**: Cung cấp HTTP API cho backend.
- **AWS Lambda**: Xử lý API, tiếp nhận tài liệu và logic pre-sign-up.
- **Amazon S3**: Lưu tài liệu PDF và asset của AWS CDK.
- **Amazon SQS**: Tách luồng upload khỏi quá trình xử lý tài liệu bất đồng bộ.
- **Amazon DynamoDB**: Lưu tài liệu, document chunk, hội thoại, tin nhắn, quiz, bài thi, lượt làm bài, bản tóm tắt và trạng thái AI job.
- **Amazon CloudWatch**: Cung cấp log, metric và alarm cho Lambda và SQS.
- **AWS IAM**: Phân quyền giữa người dùng và các dịch vụ AWS.
- **AWS CDK và AWS CloudFormation**: Khai báo và triển khai lặp lại hạ tầng staging, production.

#### Công nghệ hỗ trợ
- **GitHub**: Quản lý source code, phối hợp qua branch và cung cấp nguồn cho continuous deployment của Amplify.
- **Ollama**: Cung cấp model ngôn ngữ từ máy chủ AI local tự quản lý.
- **Qwen 2.5 7B**: Model ngôn ngữ local được sử dụng trong luồng AI.
- **Cloudflare relay**: Cung cấp lớp relay liên quan đến việc truy cập dịch vụ AI local. Cơ chế relay cụ thể cần được xác minh thêm từ cấu hình triển khai.

### 6. Thay đổi kiến trúc và các giới hạn 

 **Những thay đổi trong quá trình triển khai**

Trong quá trình phát triển, một số thành phần trong kiến trúc ban đầu đã được điều chỉnh nhằm phù hợp với điều kiện thực tế, giới hạn của tài khoản AWS và yêu cầu của dự án. Các thay đổi chính được trình bày trong bảng dưới đây.

| Đề xuất ban đầu | Phương án triển khai | Lý do |
|-----------------|----------------------|--------|
| Custom domain bằng Amazon Route 53 | Sử dụng tên miền mặc định của AWS Amplify (`*.amplifyapp.com`) | Nhóm không thể hoàn tất việc đăng ký và cấu hình tên miền riêng trong thời gian thực tập. |
| Amazon Bedrock | Triển khai mô hình Qwen2.5:7B bằng Ollama trên máy chủ AI tự quản lý | Tài khoản AWS của dự án không được cấp quyền sử dụng Amazon Bedrock và các API liên quan. |
| AWS Secrets Manager | Quản lý khóa API và thông tin cấu hình thông qua biến môi trường (Environment Variables) | Giảm chi phí triển khai và phù hợp với quy mô của dự án thực tập. |
| AWS CloudTrail | Theo dõi hệ thống bằng Amazon CloudWatch | CloudTrail không nằm trong phạm vi triển khai và không cần thiết đối với yêu cầu của dự án. |
| Xử lý tài liệu trực tiếp trong API | Sử dụng Amazon SQS làm hàng đợi xử lý | Cho phép xử lý bất đồng bộ, giảm thời gian phản hồi của API và tăng khả năng mở rộng hệ thống. |
| Một môi trường triển khai duy nhất | Tách riêng môi trường Staging và Production | Giảm rủi ro khi kiểm thử tính năng mới, tránh ảnh hưởng đến hệ thống đang vận hành. |

### 7. Kế hoạch triển khai  
- **Lập kế hoạch**: Xác định vấn đề, tính năng, user flow và kiến trúc AWS ban đầu.
- **Điều chỉnh kiến trúc**: Kiểm tra khả năng sử dụng dịch vụ và thay thế các thành phần không khả dụng.
- **Phát triển ứng dụng**: Xây dựng frontend, backend API, xử lý tài liệu, phòng học AI và bài luyện tập.
- **Triển khai hạ tầng**: Khởi tạo tài nguyên staging và production bằng AWS CDK, CloudFormation.
- **Tích hợp**: Kết nối GitHub với Amplify và tích hợp Cognito, API Gateway, Lambda, S3, SQS, DynamoDB cùng dịch vụ AI local.
- **Kiểm thử**: Kiểm tra đăng ký, upload, xử lý tài liệu, hỏi đáp, tạo quiz, chấm điểm và xem kết quả.


### 8. Rủi ro và phương án giảm thiểu
 
Trong quá trình triển khai và vận hành hệ thống, một số rủi ro có thể ảnh hưởng đến tính ổn định, hiệu năng và chi phí của dự án. Để hạn chế các tác động này, nhóm đã xác định các biện pháp giảm thiểu tương ứng như sau.

| Rủi ro | Ảnh hưởng | Phương án giảm thiểu |
|---------|-----------|----------------------|
| Máy chủ AI local hoặc dịch vụ Ollama không khả dụng | Chức năng trò chuyện với AI, tạo câu hỏi và hỗ trợ học tập có thể bị gián đoạn. | Duy trì máy chủ AI hoạt động ổn định trong quá trình trình diễn, giám sát trạng thái kết nối và định hướng chuyển sang dịch vụ AI được quản lý trên nền tảng đám mây trong tương lai. |
| Relay hoặc kết nối Internet không ổn định | Yêu cầu đến mô hình AI có thể bị chậm hoặc thất bại. | Thiết lập cơ chế timeout, xử lý ngoại lệ và cho phép người dùng thực hiện gửi lại yêu cầu khi cần thiết. |
| Quá trình xử lý tài liệu thất bại | Tài liệu không được xử lý, người dùng không thể sử dụng để học hoặc tạo câu hỏi. | Sử dụng Amazon SQS để xử lý bất đồng bộ, theo dõi trạng thái xử lý, bổ sung cơ chế xử lý lỗi trong ứng dụng và giám sát bằng Amazon CloudWatch. |
| Thay đổi trên môi trường staging ảnh hưởng đến production | Có thể gây gián đoạn dịch vụ hoặc ảnh hưởng đến dữ liệu của người dùng. | Tách biệt hoàn toàn môi trường staging và production, chỉ triển khai lên production sau khi mã nguồn được kiểm thử và hợp nhất (merge) vào nhánh chính trên GitHub. |
| Chi phí sử dụng AWS vượt dự kiến | Phát sinh chi phí ngoài kế hoạch của dự án. | Theo dõi mức sử dụng tài nguyên thông qua AWS Billing và CloudWatch, đồng thời chỉ duy trì các tài nguyên cần thiết trong suốt quá trình đánh giá và trình diễn hệ thống. |


### 9. Kết quả kỳ vọng

- Ứng dụng SmartStudy được triển khai và truy cập qua AWS Amplify.
- Đăng ký và đăng nhập an toàn bằng Amazon Cognito.
- Upload PDF và xử lý tài liệu bất đồng bộ ổn định.
- Hỏi đáp AI dựa trên tài liệu và tự động tạo bài luyện tập.
- Lưu lịch sử học tập và hỗ trợ xem lại kết quả.
- Hạ tầng AWS staging, production có thể triển khai lặp lại.
- Tạo nền tảng để thay Ollama local bằng dịch vụ AI managed trong phiên bản tương lai.
- Bản đề xuất này được xây dựng từ tài nguyên AWS đã triển khai, video demo hoàn chỉnh và tổng quan repository GitHub hiện có. - Chi tiết kết nối Cloudflare với Ollama sẽ được hoàn thiện sau khi xác minh cấu hình vận hành của máy chủ AI local.