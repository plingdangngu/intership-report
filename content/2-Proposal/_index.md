---
title: "Proposal"
date: 2026-07-20
weight: 2
chapter: false
pre: " <b> 2. </b> "
---

## SmartStudy AI - A Serverless AI-Powered Learning Assistant for PDF Documents

### 1. Executive Summary

SmartStudy AI is a web-based learning platform that enables students to study directly from their own PDF documents. Users can create an account, upload and manage documents, ask questions based on document content, generate practice quizzes, submit answers, and review scores with explanations.

The application adopts a serverless architecture on AWS for web hosting, authentication, API management, document storage, asynchronous processing, application data storage, and system monitoring. Since the project account could not access Amazon Bedrock, the AI component was implemented using the Qwen2.5:7B model running on Ollama hosted on a self-managed local AI server. The project repository also includes a Cloudflare relay component to facilitate communication with the local AI service. The deployment details will be further verified when the technical documentation is finalized.

### 2. Problem Statement

Students often need to study from multiple lengthy and disconnected PDF documents. Reading, summarizing, creating practice questions, and tracking learning progress manually is both time-consuming and inefficient. General-purpose AI chatbots may also generate responses unrelated to the uploaded documents, making it difficult to verify whether the answers are grounded in the provided learning materials.

SmartStudy addresses these challenges by providing a unified learning environment that allows users to:

- Upload and organize PDF documents.
- Ask questions based on uploaded documents.
- Receive answers with references to relevant document content.
- Automatically generate practice quizzes.
- Store conversations, quiz attempts, scores, and explanations.

### 3. Proposed Solution

SmartStudy consists of the following primary workflow:

1. Users register or sign in using Amazon Cognito.
2. The web application sends requests to the backend through Amazon API Gateway.
3. Uploaded PDF documents are stored in Amazon S3 and forwarded to Amazon SQS for asynchronous processing.
4. AWS Lambda processes the documents and stores metadata, document chunks, conversations, quizzes, and quiz attempts in Amazon DynamoDB.
5. AI requests are forwarded to the locally hosted Ollama service through the project's relay mechanism.
6. Amazon CloudWatch collects logs, metrics, and alarms for system monitoring.

### 4. Preliminary Solution Architecture

#### Initial Proposed Architecture

The following architecture represents the original system design before implementation. It is retained to illustrate the initial design approach, although several services were later replaced due to AWS account limitations and deployment constraints.

<div style="text-align: center;">
  <img src="/intership-report/images/Gallery/OLD_Diagram.jpg"
       alt="Initial SmartStudy AI Architecture"
       width="900">
</div>

<p align="center">
  <i>Figure 1. Initial proposed architecture of the SmartStudy AI system.</i>
</p>

#### Final Architecture

<div style="text-align: center;">
  <img src="/intership-report/images/Gallery/NEW_Diagram.jpg"
       alt="Final SmartStudy AI Architecture"
       width="900">
</div>

<p align="center">
  <i>Figure 2. Final deployed architecture combining AWS serverless services and a self-managed Ollama AI environment.</i>
</p>

The project maintains separate staging and production environments in the (`*.us-east-1`) region. The staging branch on GitHub is used for integration and testing. After validation, changes are merged into the main branch and automatically deployed to production through AWS Amplify.

### 5. AWS Services and Technologies

#### AWS Services

- **AWS Amplify Hosting** – Builds and deploys the frontend directly from GitHub.
- **Amazon Cognito** – Handles user registration, authentication, and token management.
- **Amazon API Gateway** – Provides HTTP APIs for backend services.
- **AWS Lambda** – Implements API logic, document processing, and Cognito pre-sign-up triggers.
- **Amazon S3** – Stores uploaded PDF documents and AWS CDK deployment assets.
- **Amazon SQS** – Enables asynchronous document processing.
- **Amazon DynamoDB** – Stores documents, document chunks, conversations, messages, quizzes, quiz attempts, summaries, and AI job status.
- **Amazon CloudWatch** – Provides logs, metrics, and alarms for Lambda and SQS.
- **AWS IAM** – Manages permissions among AWS services.
- **AWS CDK & AWS CloudFormation** – Provision and deploy staging and production infrastructure as code.

#### Supporting Technologies

- **GitHub** – Source code management, branch collaboration, and deployment source for AWS Amplify.
- **Ollama** – Hosts the local large language model.
- **Qwen2.5:7B** – Local language model used for AI-powered learning features.
- **Cloudflare Relay** – Provides secure connectivity between AWS services and the locally hosted AI server.

### 6. Architecture Changes and Deployment Constraints

During development, several architectural components were modified to better accommodate practical deployment constraints and AWS account limitations.

| Initial Proposal | Final Implementation | Reason |
|------------------|----------------------|--------|
| Custom domain via Amazon Route 53 | AWS Amplify default domain (`*.amplifyapp.com`) | The team was unable to complete domain registration during the internship. |
| Amazon Bedrock | Ollama running the Qwen2.5:7B model on a self-managed local AI server | The project AWS account was not authorized to access Amazon Bedrock services. |
| AWS Secrets Manager | Environment variables | Reduced deployment complexity and cost for the internship project. |
| AWS CloudTrail | Amazon CloudWatch | CloudTrail was outside the implementation scope. |
| Direct document processing | Amazon SQS processing queue | Asynchronous processing improves scalability and reduces API response time. |
| Single deployment environment | Separate Staging and Production environments | Reduces deployment risk and protects production resources during testing. |

### 7. Implementation Plan

- **Planning** – Define project objectives, features, user flow, and initial AWS architecture.
- **Architecture Adjustment** – Replace unavailable AWS services with practical alternatives.
- **Application Development** – Build the frontend, backend APIs, document processing pipeline, AI learning module, and quiz system.
- **Infrastructure Deployment** – Provision staging and production environments using AWS CDK and CloudFormation.
- **System Integration** – Integrate GitHub, Amplify, Cognito, API Gateway, Lambda, S3, SQS, DynamoDB, and the local AI service.
- **Testing** – Verify user registration, authentication, document upload, document processing, AI chat, quiz generation, grading, and result visualization.

### 8. Risks and Mitigation Strategies

Potential operational risks and corresponding mitigation strategies are summarized below.

| Risk | Impact | Mitigation |
|------|--------|------------|
| Local AI server or Ollama becomes unavailable | AI chat and quiz generation become unavailable | Maintain server availability during demonstrations, continuously monitor connectivity, and plan migration to a managed AI service in future versions. |
| Relay service or Internet instability | AI requests become slow or fail | Configure request timeouts, exception handling, and user-friendly retry mechanisms. |
| Document processing failure | Uploaded documents cannot be used for learning | Use Amazon SQS for asynchronous processing, track processing status, implement error handling, and monitor with Amazon CloudWatch. |
| Staging changes affecting production | Service interruption or data inconsistency | Isolate staging and production resources and deploy only after successful testing and GitHub merge. |
| AWS resource usage exceeds expectations | Unexpected cloud costs | Monitor resource usage through AWS Billing and CloudWatch while keeping only essential resources active during evaluation.

### 9. Expected Outcomes

- SmartStudy is successfully deployed and accessible through AWS Amplify.
- Secure user authentication using Amazon Cognito.
- Reliable asynchronous PDF upload and document processing.
- AI-powered question answering and automatic quiz generation based on uploaded documents.
- Persistent learning history and quiz results.
- Repeatable infrastructure deployment for staging and production environments.
- A flexible architecture that enables future migration from Ollama to a managed AI service.
- This proposal is based on the deployed AWS infrastructure, the completed project demonstration, and the current GitHub repository. The Cloudflare relay configuration for the local Ollama server will be finalized after deployment verification.