---
title: "Blog 1"
date: 2026-07-20
weight: 1
chapter: false
pre: " <b> 3.1. </b> "
---


# BUILDING AN AI GATEWAY FOR AMAZON BEDROCK USING AMAZON API GATEWAY

As Generative AI applications are increasingly being deployed in enterprises, managing access to AI models has become an important requirement. In addition to ensuring that users are granted the appropriate permissions, organizations also need to control usage quotas, isolate data between different user groups (tenant isolation), and optimize costs when using models on Amazon Bedrock. To address these requirements, Dynatrace developed an AI Gateway that acts as an intermediary layer between applications and Amazon Bedrock. This architecture was later developed into a reference architecture that enables organizations to easily deploy and manage access to AI services at scale.

In this solution, Amazon API Gateway serves as the entry point for all requests from applications before they are forwarded to Amazon Bedrock. The AI Gateway supports many important features, including user authentication through JWT or existing identity systems, quota management and request throttling, API lifecycle management, canary release deployment, and integration with AWS WAF to enhance security. In addition, the architecture supports Response Streaming, allowing responses from AI models to be streamed to users in real time as content is generated, thereby improving the user experience.

<div style="text-align: center;">
  <img src="/intership-report/images/Blog/3.1/1.png"
       alt="AI Gateway Architecture"
       width="900">
</div>

<p align="center">
  <i>Figure 1. Reference architecture of the AI Gateway using Amazon API Gateway and Amazon Bedrock.</i>
</p>

### Main Components of the AI Gateway

The AI Gateway architecture consists of four core components, each responsible for a specific role in receiving, authenticating, and forwarding requests to Amazon Bedrock.

**Amazon Route 53**

Amazon Route 53 is used to configure a custom domain name for the AI Gateway. Instead of using the default address provided by Amazon API Gateway, organizations can create an endpoint with their own domain name, making access more convenient and giving the system a more professional appearance.

**Amazon API Gateway**

Amazon API Gateway acts as the entry point for all requests from applications. It is the central component of the AI Gateway, responsible for handling tasks such as user authentication, request throttling, API lifecycle management, and integration with other AWS security services.

By using API Gateway, all requests are centrally managed before being forwarded to Amazon Bedrock, making it easier for organizations to control the use of AI models.

**AWS Lambda Authorizer**

After a request is sent to API Gateway, AWS Lambda Authorizer performs user authentication and authorization. In Dynatrace's reference architecture, Lambda Authorizer uses JWT (JSON Web Token) to verify user identities through the existing authentication system.

Depending on deployment requirements, users can also replace it with other authentication methods such as Amazon Cognito User Pools, Single Sign-On (SSO), or custom authentication mechanisms developed by the organization.

**Lambda Integration**

Lambda Integration is responsible for forwarding requests to Amazon Bedrock. After receiving a request from API Gateway, the Lambda function preserves the entire request, including its headers, body, and associated parameters.

Next, Lambda signs the request using AWS Signature Version 4 (SigV4) and forwards it to the appropriate Amazon Bedrock endpoint.

By preserving the request structure, the AI Gateway can support both existing APIs and new APIs introduced by Amazon Bedrock in the future without requiring changes to the Gateway source code, significantly reducing maintenance effort.

**Amazon Bedrock**

Amazon Bedrock is AWS's generative AI service that provides access to multiple Foundation Models and AI capabilities such as text generation, chatbots, Knowledge Bases, and AI Agents.

After receiving the request from Lambda Integration, Amazon Bedrock processes it and returns the result to the AI Gateway, which then sends the response back to the application.

#### Benefits of the AI Gateway Architecture

One of the greatest advantages of this architecture is its transparency to client applications. Applications can continue using AWS SDKs, such as Boto3 for Python, to invoke Amazon Bedrock APIs exactly as they would when connecting directly to the service.

Meanwhile, all functions such as authentication, authorization, quota management, request throttling, and security mechanisms are handled behind the scenes by the AI Gateway. This reduces the application's dependency on the underlying infrastructure and makes it easier to scale or modify the architecture in the future.

**Request Processing Workflow**: When an application sends a request to the AI Gateway to access Amazon Bedrock, the system performs the following steps:

1. The AI Gateway receives the complete request from the application, including headers, content, and related parameters.
2. Lambda Integration preserves the request information and signs it using AWS Signature Version 4.
3. After authentication, the request is forwarded to the appropriate Amazon Bedrock endpoint.
4. Amazon Bedrock processes the request and returns the result to the AI Gateway.
5. The AI Gateway returns the response to the application.

Thanks to this mechanism, the AI Gateway does not need to understand the details of individual Amazon Bedrock APIs. When AWS introduces new features or APIs, the Gateway can continue forwarding requests without requiring source code updates, reducing maintenance costs and ensuring long-term scalability.

## Deploying the AI Gateway on AWS

After understanding the architecture and workflow of the AI Gateway, the next step is to deploy the system on AWS. AWS provides sample source code together with an AWS CloudFormation template, enabling users to quickly create all the required infrastructure without manually configuring each service.

The deployment process creates components such as Amazon API Gateway, AWS Lambda Authorizer, Lambda Integration, IAM Roles, and other related resources. After deployment is complete, the system generates an API Gateway endpoint that applications can use instead of the default Amazon Bedrock endpoint.

**Step 1: Prepare the Environment**

Before deployment, ensure that the AWS account has permission to use Amazon Bedrock and has sufficient permissions to create services such as Amazon API Gateway, AWS Lambda, IAM, and AWS CloudFormation.

AWS provides the complete AI Gateway source code on GitHub so users can download and deploy it.

```bash
git clone https://github.com/aws-samples/sample-ai-gateway-for-amazon-bedrock.git

cd sample-ai-gateway-for-amazon-bedrock
```

After downloading the source code, users can modify the configuration parameters before deploying the solution to AWS.

---

**Step 2: Deploy Using AWS CloudFormation**

The project uses AWS CloudFormation to automatically provision all required infrastructure. Users only need to deploy the template instead of manually creating each service.

The following example demonstrates how to deploy the stack using the AWS CLI.

```bash
aws cloudformation deploy \
    --template-file infrastructure/template.yaml \
    --stack-name ai-gateway \
    --capabilities CAPABILITY_IAM
```

After the deployment process is complete, CloudFormation creates resources such as Amazon API Gateway, Lambda Authorizer, Lambda Integration, and IAM Roles. The system also returns the API Gateway endpoint, allowing applications to start using the AI Gateway.

---
**Step 3: Test the AI Gateway**

After successful deployment, applications can continue using the AWS SDK as usual. The only difference is that the endpoint is replaced with the AI Gateway endpoint.

The example below uses the Boto3 library to send requests to Amazon Bedrock through the AI Gateway.

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

In the example above, the application continues to invoke the API through the AWS SDK just as it would when communicating directly with Amazon Bedrock. However, all requests are first routed to the AI Gateway, where authentication, authorization, quota management, and AWS Signature Version 4 request signing are performed before the requests are forwarded to Amazon Bedrock.

---

## Response Streaming

One of the key features of the AI Gateway is its support for Response Streaming.

Normally, when invoking an AI model, users must wait until the entire response has been generated before receiving any output. This can increase waiting time, especially for large responses.

The AI Gateway supports Response Streaming, allowing Amazon Bedrock to stream generated content incrementally as it becomes available. This enables applications to receive data continuously and display responses to users almost in real time.

The following example demonstrates how to use the Response Streaming API.

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

With this mechanism, the user experience in chatbot and AI assistant applications is significantly improved because responses appear while the model is still generating them.

---

## Possible Enhancements

The AI Gateway architecture is designed to be extensible, making it easy to add new capabilities to meet real-world enterprise requirements.

Some enhancements recommended by AWS include:

- Integrating AWS WAF to protect APIs from malicious requests.
- Using API Keys and Usage Plans to manage quotas for different user groups.
- Adding a caching mechanism to reduce calls to Amazon Bedrock and optimize costs.
- Integrating Amazon CloudWatch to monitor Gateway activities.
- Applying Canary Deployment when releasing new versions.
- Managing multiple API versions through API Versioning.

Because the architecture is serverless, these components can be added without affecting client applications.

---

## Advantages and Limitations

### Advantages

- Centralized management of Amazon Bedrock access.
- Supports user authentication and authorization.
- Controls quotas and request rate limits.
- Easily integrates with existing identity management systems.
- Supports Response Streaming.
- Highly scalable through a serverless architecture.
- Rapid deployment using AWS CloudFormation.

### Limitations

- The architecture is more complex than calling Amazon Bedrock directly.
- Requires proper configuration of IAM and Lambda Authorizer.
- May incur additional costs when using services such as AWS WAF or Amazon CloudWatch.

---

## Conclusion

The AI Gateway is a reference architecture that enables organizations to centrally manage access to Amazon Bedrock. By leveraging Amazon API Gateway and AWS Lambda, the system provides authentication, authorization, quota management, and protection for AI models without requiring significant changes to client applications.

In addition, its support for Response Streaming, deployment through AWS CloudFormation, and seamless integration with other AWS services make the AI Gateway a suitable solution for modern Generative AI systems.

---

**References**: https://aws.amazon.com/blogs/architecture/building-an-ai-gateway-to-amazon-bedrock-with-amazon-api-gateway/