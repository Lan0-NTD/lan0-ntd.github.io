---
title: "Proposal"
weight: 2
pre: " <b> 1. </b> "
---



# AI Contract Intelligence SaaS

## AGREEME

Automated Contract Analysis, Risk Detection & Suggestion System

---

## 1. Summary

This document presents a SaaS solution that leverages AI models to assist small and medium-sized enterprises (SMEs) and individual users in reading, analyzing, and automatically generating contracts. The system uses OCR to extract clauses from user-submitted images or documents, after which the AI model detects potential risks and suggests more favorable modifications for the user.

The solution is deployed on the AWS platform using EC2, n8n, DynamoDB, Lambda, and Amplify, ensuring comprehensive security, modularity, and scalability.

---

## 2. Problem Statement

### What’s the Problem?

- SMEs lack in-house legal expertise for reviewing contracts.  
- Manual contract reviews are slow, inconsistent, and prone to human error.  
- Legal consulting costs are high, preventing scalable document evaluation.

### The Solution

An AI-assisted SaaS platform that:

- Extracts and classifies contract clauses automatically.  
- Highlights potential legal risks with citation context.  
- Suggests negotiation points and generates revised versions.  
- Supports template-based contract drafting.

### Benefits and Return on Investment

| Category | Description |
|-----------|--------------|
| Efficiency | Reduces contract review time from hours to minutes |
| Cost Saving | Lowers legal consulting cost by 60–70% |
| Scalability | SaaS deployment allows multi-tenant scaling |
| Compliance | Ensures consistent risk evaluation and legal citation tracking |

---

## 3. Solution Architecture

![System Architecture Diagram](Pipeline.png)


The system is orchestrated by AWS EC2, with automation through n8n workflows. All AI inference and OCR are handled by external APIs (OpenAI + DeepSeekOCR).

### AWS Services Used

| No. | AWS Service | Function |
|--------|--------------|----------|
| 1 | Amazon Route 53 | Receives requests from Users and routes traffic. It points to both the Frontend (Amplify) and Backend (API Gateway). |
| 2 | Amazon WAF | (Frontend Flow) A web application firewall service that filters malicious requests before they reach the frontend. |
| 3 | Amazon Amplify | (Frontend Flow) A hosting, build, and deployment service for web (frontend) applications for users. |
| 4 | Amazon API Gateway | (Backend Flow) Receives API requests from users and serves as the “gateway” for all backend business logic. |
| 5 | Amazon Cognito | Called by the API Gateway to authenticate and authorize users, ensuring that only valid users can access the API. |
| 6 | AWS Lambda (Invoke) | Triggered by the API Gateway. This Lambda function performs a quick task — in this case, it invokes and starts the main workflow. |
| 7 | EC2 | Runs n8n — an orchestration service that receives commands from the API Gateway to initiate and manage data-processing workflows and the embedding service (a sub-service called by n8n that performs data vectorization). Another EC2 instance is used to host the SQL user database. |
| 8 | Amazon S3 Raw | Object storage service. |
| 9 | Amazon DynamoDB | NoSQL database service. |
| 10 | AWS Lambda (Invoke) | The n8n service calls another Lambda function to execute specific logic (in this diagram, it performs an external API call). |
| 11 | External LLM APIs | The Lambda function calls a third-party API (e.g., OpenAI, Claude) for language processing. |
| 12 | Amazon Secrets Manager | The EC2 instance accesses this service to securely retrieve sensitive information (such as API keys or database passwords). |
| 13 | Amazon CloudWatch | Monitoring and logging service. Lambda functions (and potentially other services) send logs and metrics here for operational tracking. |
| 14 | GitLab | The GitLab CI/CD system automatically triggers upon code changes to build and deploy the application to Amplify. |
| 15 | Amazon S3 Web | Amazon Amplify uses this S3 bucket to store static web files (HTML, JS, CSS) after the application build process. |
### Component Design

- Frontend (Amplify + S3) – UI for contract upload, risk visualization, and report generation.  
- API Gateway + Cognito – Authentication and API access management.  
- EC2 (n8n Orchestrator) – Runs workflow templates (WT-01 → WT-07), uploads and normalizes files to S3.  
- RDS + DynamoDB – Store relational metadata and embeddings.  
- AI API Integrations – Provide text extraction and reasoning logic.  
- CloudWatch + Secrets Manager – Monitor health, rotate keys, and secure credentials.

---

## 4. Technical Implementation

### 4.1 Implementation Phases

**MVP (Local)**

- n8n environment hosted with EC2 instance.  
- Integration with external LLM APIs for OCR + contract analysis.  
- Local DynamoDB instance for embedding storage.  
- Manual API gateway mock for testing workflow end-to-end.

**Cloud Integration**

- Amplify connected to GitLab repository for CI/CD pipeline.  
- Configure Aurora (PostgreSQL) and DynamoDB for production.  
- Integrate API Gateway + Cognito.  
- Enable CloudWatch logging and WAF security layer.

---

### 4.2 Optimization For Production

- Utilize RDS read replicas to offload analytical queries.  
- Optimize embedding retrieval with DynamoDB on-demand capacity.  
- Cache intermediate LLM outputs to S3 for reusability.  
- Enable autoscaling on EC2 for workflow workloads.  
- Adopt CloudWatch alarms + notification through n8n hooks.

---

### 4.3 Technical Requirements

| Category | Specification |
|-----------|---------------|
| Programming Stack | n8n workflows, Net.js 15 + TailwindCSS (Amplify) |
| External APIs | DeepSeek OCR, GPT-5 |
| Storage | RDS PostgreSQL, DynamoDB, S3 |
| Orchestration | EC2 instance (t4g.large) |
| Security | WAF, Secrets Manager, Cognito |
| Monitoring | CloudWatch + n8n Alert Hooks |

---

## 5. Timeline & Milestones

| Phase | Deliverable | Duration |
|--------|--------------|-----------|
| 1 | MVP backend (n8n + DynamoDB + OCR pipeline) | 3 weeks |
| 2 | Contract risk analysis (GPT integration) | 3 weeks |
| 3 | Revision & template generator | 2 weeks |
| 4 | Full AWS deployment (Amplify + Cognito + EC2) | 3 weeks |
| 5 | Monitoring, scaling, and pilot test | 2 weeks |

---

## 6. Budget Estimation

| Component | AWS Service | Monthly Cost (USD) | Notes |
|------------|-------------|--------------------|--------|
| Frontend Hosting | Amplify | 3.68 |  |
| DNS/Routing | Route 53 | 2.04 |  |
| Backend Compute | EC2 | 16.12 |  |
| User Database | RDS for PostgreSQL | 8.73 |  |
| Vector Database | DynamoDB | 4.02 |  |
| Storage | S3 | 1.93 |  |
| Request Routing | API Gateway | 1.29 |  |
| Virtual Cloud | VPC | 38.01 |  |
| AI API Usage | OpenAI | 5.00 | Non-AWS |
| User Authentication | Cognito | 1.00 |  |
| Security | Secrets Manager | 0.94 |  |
| Security | WAF | 6.60 |  |
| Monitoring | CloudWatch | 0.53 |  |
| **Total Estimate** |  | **78.73** | **~255/13 weeks** |

### Cost Optimization

Most components use serverless and managed AWS services, minimizing idle resources and operational overhead.

Compute: EC2 uses Graviton t4g.large with limited runtime (12h/day).

Database: RDS t4g.micro and DynamoDB (50 GB) are right-sized for dev/test load, avoiding over-provisioning.

Storage: Separate S3 buckets for raw data and web hosting keep data organized and cost-controlled.

Integration & Security: Use of HTTP API Gateway, single WAF rule, and one Route 53 hosted zone reduces costs.

Overall monthly cost is around 65 USD, reflecting a well-optimized deployment.

---

## 7. Risk Assessment

| Risk | Likelihood | Mitigation |
|------|-------------|-------------|
| High API costs for LLM usage | Medium | Use caching, partial context retrieval |
| Data privacy concerns | Low | Encrypt S3 and Aurora data with KMS |
| Workflow errors or timeouts | Medium | n8n retry logic + CloudWatch alarms |
| Vendor API change (OpenAI/DeepSeek) | Low | API abstraction layer for replacement |
| Performance under heavy load | High | EC2 autoscaling and Aurora read replicas |

---

## 8. Expected Outcomes

### 8.1 Technical Improvements

- End-to-end contract analytics pipeline deployed fully on AWS.  
- Modular backend orchestration via n8n on EC2.  
- Secure, CI/CD-driven workflow through Amplify.  
- Scalable data handling with RDS + DynamoDB.

### 8.2 Long-term Value

- Enables SMEs to adopt AI-assisted contract intelligence affordably.  
- Reduces dependency on external legal resources.  
- Establishes groundwork for enterprise-level contract governance tools.
