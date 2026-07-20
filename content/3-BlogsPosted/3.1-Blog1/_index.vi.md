---
title: "Blog 1"
date: 2026-07-20
weight: 1
chapter: false
pre: " <b> 3.1. </b> "
---


# XÂY DỰNG AI GATEWAY CHO AMAZON BEDROCK BẰNG AMAZON API GATEWAY

Khi các ứng dụng Generative AI ngày càng được triển khai trong doanh nghiệp, việc quản lý quyền truy cập đến các mô hình AI trở thành một yêu cầu quan trọng. Bên cạnh việc đảm bảo người dùng được cấp đúng quyền, doanh nghiệp còn cần kiểm soát hạn mức sử dụng, phân tách dữ liệu giữa các nhóm người dùng (tenant isolation) và tối ưu chi phí khi khai thác các mô hình trên Amazon Bedrock.Để đáp ứng những yêu cầu đó, Dynatrace đã xây dựng một mô hình AI Gateway hoạt động như một lớp trung gian giữa ứng dụng và Amazon Bedrock. Kiến trúc này sau đó được phát triển thành một mô hình tham chiếu (reference architecture) giúp các tổ chức dễ dàng triển khai và quản lý việc truy cập các dịch vụ AI trên quy mô lớn.

Trong giải pháp này, Amazon API Gateway được sử dụng làm điểm tiếp nhận tất cả các yêu cầu từ ứng dụng trước khi chuyển đến Amazon Bedrock. AI Gateway hỗ trợ nhiều tính năng quan trọng như xác thực người dùng thông qua JWT hoặc hệ thống định danh hiện có, quản lý quota và giới hạn tốc độ truy cập (throttling), hỗ trợ quản lý vòng đời API, triển khai canary release và tích hợp với AWS WAF để tăng cường bảo mật. Ngoài ra, kiến trúc còn hỗ trợ Response Streaming, cho phép phản hồi từ mô hình AI được gửi đến người dùng theo thời gian thực ngay trong quá trình sinh nội dung, giúp cải thiện trải nghiệm sử dụng.

<div style="text-align: center;">
  <img src="/intership-report/images/Blog/3.1/1.png"
       alt="Kiến trúc AI Gateway"
       width="900">
</div>

<p align="center">
  <i>Hình 1. Kiến trúc tham chiếu của AI Gateway sử dụng Amazon API Gateway và Amazon Bedrock.</i>
</p>

### Các thành phần chính của AI Gateway

Kiến trúc AI Gateway được xây dựng từ bốn thành phần cốt lõi, mỗi thành phần đảm nhận một vai trò riêng trong quá trình tiếp nhận, xác thực và chuyển tiếp yêu cầu đến Amazon Bedrock.

**Amazon Route 53**

Amazon Route 53 được sử dụng để cấu hình tên miền riêng cho AI Gateway. Thay vì sử dụng địa chỉ mặc định do Amazon API Gateway cung cấp, doanh nghiệp có thể tạo một endpoint mang tên miền của riêng mình, giúp việc truy cập thuận tiện hơn và tăng tính chuyên nghiệp cho hệ thống.

**Amazon API Gateway**

Amazon API Gateway đóng vai trò là điểm tiếp nhận toàn bộ yêu cầu từ các ứng dụng. Đây là thành phần trung tâm của AI Gateway, chịu trách nhiệm xử lý các tác vụ như xác thực người dùng, giới hạn tốc độ truy cập (request throttling), quản lý vòng đời API và tích hợp với các dịch vụ bảo mật khác của AWS.

Nhờ sử dụng API Gateway, mọi yêu cầu đều được quản lý tập trung trước khi chuyển đến Amazon Bedrock, giúp doanh nghiệp dễ dàng kiểm soát việc sử dụng các mô hình AI.

**AWS Lambda Authorizer**

Sau khi yêu cầu được gửi đến API Gateway, AWS Lambda Authorizer sẽ thực hiện bước xác thực và phân quyền người dùng. Trong kiến trúc tham chiếu của Dynatrace, Lambda Authorizer sử dụng JWT (JSON Web Token) để xác minh danh tính thông qua hệ thống xác thực hiện có.

Tùy theo nhu cầu triển khai, người dùng cũng có thể thay thế bằng các phương thức xác thực khác như Amazon Cognito User Pools, hệ thống Single Sign-On (SSO) hoặc các cơ chế xác thực do doanh nghiệp tự xây dựng.

**Lambda Integration**

Lambda Integration là thành phần chịu trách nhiệm chuyển tiếp yêu cầu đến Amazon Bedrock. Sau khi nhận request từ API Gateway, hàm Lambda sẽ giữ nguyên toàn bộ thông tin của yêu cầu, bao gồm header, nội dung (body) và các tham số đi kèm.

Tiếp theo, Lambda sẽ ký request bằng cơ chế AWS Signature Version 4 (SigV4) và chuyển tiếp đến đúng endpoint của Amazon Bedrock.

Nhờ giữ nguyên cấu trúc của request, AI Gateway có thể hỗ trợ cả các API hiện tại lẫn những API mới được Amazon Bedrock bổ sung trong tương lai mà không cần sửa đổi mã nguồn của Gateway, giúp giảm đáng kể công sức bảo trì hệ thống.

**Amazon Bedrock**

Amazon Bedrock là dịch vụ AI tạo sinh của AWS, cung cấp quyền truy cập đến nhiều Foundation Models và các tính năng AI như sinh văn bản, chatbot, Knowledge Bases hay AI Agents.

Sau khi nhận request từ Lambda Integration, Amazon Bedrock sẽ xử lý yêu cầu và trả kết quả về AI Gateway để phản hồi lại cho ứng dụng.

#### Lợi ích của kiến trúc AI Gateway

Một trong những ưu điểm lớn nhất của kiến trúc này là tính minh bạch đối với ứng dụng phía client. Các ứng dụng vẫn có thể sử dụng các SDK của AWS (chẳng hạn như Boto3 đối với Python) để gọi các API của Amazon Bedrock giống như khi kết nối trực tiếp đến dịch vụ.

Trong khi đó, toàn bộ các chức năng như xác thực, phân quyền, quản lý quota, giới hạn tốc độ truy cập và các cơ chế bảo mật đều được AI Gateway xử lý phía sau. Điều này giúp giảm sự phụ thuộc của ứng dụng vào hạ tầng phía dưới và tạo điều kiện thuận lợi khi mở rộng hoặc thay đổi kiến trúc trong tương lai.

**Quy trình xử lý yêu cầu**: Khi một ứng dụng gửi yêu cầu đến AI Gateway để sử dụng Amazon Bedrock, hệ thống sẽ thực hiện các bước sau:

1. AI Gateway tiếp nhận toàn bộ request từ ứng dụng, bao gồm header, nội dung và các tham số liên quan.
2. Lambda Integration giữ nguyên thông tin của request và thực hiện ký yêu cầu bằng AWS Signature Version 4.
3. Request sau khi được xác thực sẽ được chuyển tiếp đến đúng endpoint của Amazon Bedrock.
4. Amazon Bedrock xử lý yêu cầu và trả kết quả về AI Gateway.
5. AI Gateway phản hồi kết quả lại cho ứng dụng.


Nhờ cơ chế này, AI Gateway không cần biết chi tiết từng API của Amazon Bedrock. Khi AWS bổ sung các tính năng hoặc API mới, Gateway vẫn có thể chuyển tiếp request mà không cần cập nhật mã nguồn, giúp giảm chi phí bảo trì và đảm bảo khả năng mở rộng lâu dài cho hệ thống.

## Triển khai AI Gateway trên AWS

Sau khi tìm hiểu kiến trúc và quy trình hoạt động của AI Gateway, bước tiếp theo là triển khai hệ thống trên AWS. AWS cung cấp sẵn mã nguồn mẫu cùng với AWS CloudFormation Template, giúp người dùng nhanh chóng tạo toàn bộ hạ tầng cần thiết mà không phải cấu hình thủ công từng dịch vụ.

Quá trình triển khai sẽ tạo các thành phần như Amazon API Gateway, AWS Lambda Authorizer, Lambda Integration, IAM Role và các tài nguyên liên quan. Sau khi hoàn tất, hệ thống sẽ sinh ra một API Gateway Endpoint để các ứng dụng sử dụng thay cho endpoint mặc định của Amazon Bedrock.

**Bước 1: Chuẩn bị môi trường**

Trước khi triển khai, cần đảm bảo tài khoản AWS đã được cấp quyền sử dụng Amazon Bedrock và có quyền tạo các dịch vụ như Amazon API Gateway, AWS Lambda, IAM và AWS CloudFormation.

AWS cung cấp sẵn toàn bộ mã nguồn của AI Gateway trên GitHub để người dùng có thể tải về và triển khai.

```bash
git clone https://github.com/aws-samples/sample-ai-gateway-for-amazon-bedrock.git

cd sample-ai-gateway-for-amazon-bedrock
```

Sau khi tải mã nguồn, người dùng có thể chỉnh sửa các thông số cấu hình trước khi triển khai lên AWS.

---

**Bước 2: Triển khai bằng AWS CloudFormation**

Dự án sử dụng AWS CloudFormation để tự động tạo toàn bộ hạ tầng cần thiết. Người dùng chỉ cần triển khai template thay vì tạo thủ công từng dịch vụ.

Ví dụ dưới đây minh họa cách triển khai stack bằng AWS CLI.

```bash
aws cloudformation deploy \
    --template-file infrastructure/template.yaml \
    --stack-name ai-gateway \
    --capabilities CAPABILITY_IAM
```

Sau khi quá trình triển khai hoàn tất, CloudFormation sẽ tạo các tài nguyên như Amazon API Gateway, Lambda Authorizer, Lambda Integration và IAM Roles. Đồng thời hệ thống cũng trả về địa chỉ API Gateway Endpoint để ứng dụng bắt đầu sử dụng.

---

**Bước 3: Kiểm thử AI Gateway**

Sau khi triển khai thành công, ứng dụng vẫn có thể sử dụng AWS SDK như bình thường. Điểm khác biệt duy nhất là endpoint được thay bằng địa chỉ của AI Gateway.

Ví dụ dưới đây sử dụng thư viện Boto3 để gửi yêu cầu đến Amazon Bedrock thông qua AI Gateway.

```python
import boto3
from botocore.config import Config

client = boto3.client(
    service_name="bedrock-runtime",
    endpoint_url="https://your-api-id.execute-api.us-east-1.amazonaws.com/prod",
    config=Config(retries={"max_attempts": 3})
)

response = client.converse(
    modelId="amazon.nova-lite-v1:0",
    messages=[
        {
            "role": "user",
            "content": [
                {
                    "text": "Explain Amazon Bedrock."
                }
            ]
        }
    ]
)

print(response["output"]["message"]["content"][0]["text"])
```

Trong ví dụ trên, ứng dụng vẫn gọi API thông qua AWS SDK giống như khi làm việc trực tiếp với Amazon Bedrock. Tuy nhiên, mọi yêu cầu đều được chuyển đến AI Gateway trước để thực hiện xác thực, phân quyền, quản lý quota và ký yêu cầu bằng AWS Signature Version 4 trước khi chuyển tiếp đến Amazon Bedrock.

---

## Response Streaming

Một trong những tính năng nổi bật của AI Gateway là hỗ trợ Response Streaming.

Thông thường, khi gọi mô hình AI, người dùng phải chờ đến khi toàn bộ câu trả lời được tạo xong mới nhận được phản hồi. Điều này có thể làm tăng thời gian chờ đối với những nội dung có kích thước lớn.

AI Gateway hỗ trợ cơ chế Response Streaming, cho phép Amazon Bedrock gửi từng phần nội dung ngay khi được sinh ra. Điều này giúp ứng dụng nhận được dữ liệu liên tục và hiển thị phản hồi gần như theo thời gian thực.

Ví dụ dưới đây minh họa cách sử dụng API Response Streaming.

```python
response = client.converse_stream(
    modelId="amazon.nova-lite-v1:0",
    messages=[
        {
            "role": "user",
            "content": [
                {
                    "text": "Tell me about Amazon Bedrock."
                }
            ]
        }
    ]
)

for event in response["stream"]:
    print(event)
```

Nhờ cơ chế này, trải nghiệm người dùng trong các ứng dụng chatbot hoặc trợ lý AI được cải thiện đáng kể vì câu trả lời xuất hiện ngay trong quá trình mô hình xử lý.

---

## Các hướng mở rộng

Kiến trúc AI Gateway được thiết kế theo hướng mở nên có thể dễ dàng mở rộng thêm nhiều tính năng phục vụ nhu cầu thực tế của doanh nghiệp.

Một số hướng mở rộng được AWS đề xuất bao gồm:

- Tích hợp AWS WAF để bảo vệ API khỏi các request độc hại.
- Sử dụng API Key và Usage Plan nhằm quản lý quota của từng nhóm người dùng.
- Bổ sung cơ chế Cache để giảm số lần gọi đến Amazon Bedrock và tối ưu chi phí.
- Tích hợp Amazon CloudWatch để giám sát hoạt động của Gateway.
- Áp dụng Canary Deployment khi triển khai phiên bản mới.
- Quản lý nhiều phiên bản API bằng API Versioning.

Nhờ sử dụng kiến trúc serverless, các thành phần này có thể được bổ sung mà không ảnh hưởng đến ứng dụng phía client.

---

## Ưu điểm và hạn chế

### Ưu điểm

- Quản lý tập trung việc truy cập Amazon Bedrock.
- Hỗ trợ xác thực và phân quyền người dùng.
- Kiểm soát quota và giới hạn tốc độ truy cập.
- Dễ dàng tích hợp với các hệ thống định danh hiện có.
- Hỗ trợ Response Streaming.
- Có khả năng mở rộng linh hoạt nhờ kiến trúc serverless.
- Triển khai nhanh bằng AWS CloudFormation.

### Hạn chế

- Kiến trúc phức tạp hơn so với việc gọi trực tiếp Amazon Bedrock.
- Cần cấu hình IAM và Lambda Authorizer chính xác.
- Có thể phát sinh thêm chi phí khi sử dụng các dịch vụ như WAF hoặc CloudWatch.

---

## Kết luận

AI Gateway là một kiến trúc tham chiếu giúp doanh nghiệp quản lý tập trung việc truy cập Amazon Bedrock. Thông qua Amazon API Gateway và AWS Lambda, hệ thống có thể thực hiện xác thực, phân quyền, quản lý quota và bảo vệ các mô hình AI mà không yêu cầu thay đổi lớn ở phía ứng dụng.

Bên cạnh đó, khả năng hỗ trợ Response Streaming, triển khai bằng CloudFormation và dễ dàng mở rộng với các dịch vụ khác của AWS giúp AI Gateway trở thành một giải pháp phù hợp cho các hệ thống Generative AI hiện đại.

---
**Nguồn tham khảo**: https://aws.amazon.com/vi/blogs/architecture/building-an-ai-gateway-to-amazon-bedrock-with-amazon-api-gateway/