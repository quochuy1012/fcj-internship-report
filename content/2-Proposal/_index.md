---
title: "Proposal"
date: 2024-01-01
weight: 2
chapter: false
pre: " <b> 2. </b> "
---

# SportWear E-Commerce Platform
## Sportswear e-commerce website deployed on AWS

### 1. Executive Summary

**SportWear** is a sportswear e-commerce website that lets users browse products, add items to the cart, place orders, and upload product images.

The platform is deployed on AWS with a clear separation between **frontend** (S3 + CloudFront) and **backend** (ECS Fargate behind an Application Load Balancer). Users resolve the domain with **Route 53**. **AWS WAF** is associated with the **CloudFront** distribution for basic web protection, while CloudFront accelerates and caches the frontend. Sensitive configuration is stored in **Secrets Manager**. Docker images are stored in **Amazon ECR**. The database uses a managed service (**Amazon DocumentDB**, MongoDB-compatible). Supporting services are reached privately via **VPC endpoints** (Interface/PrivateLink for ECR, Secrets Manager, CloudWatch; Gateway endpoint for S3).

### 2. Problem Statement

**Problem**
- Small sportswear shops often lack a scalable online store for catalog, cart, checkout, and image management.
- Hosting everything on a single public server is hard to secure, scale, and monitor.

**Solution**
- Build SportWear as a cloud-native e-commerce site on AWS.
- Static frontend on **S3** distributed by **CloudFront**.
- Dynamic APIs on **ECS Fargate** in **private subnets**, fronted by **ALB**.
- Managed database with **DocumentDB**.
- High availability with **2 Availability Zones**, each having **Public** and **Private** subnets.
- Private service access via **VPC endpoints** (PrivateLink for ECR / Secrets Manager / CloudWatch; Gateway for S3).

**Benefits**
- Faster page loads via CDN.
- Better security: backend has no public IP.
- Easier operations with managed DB, Secrets Manager, and CloudWatch.
- Architecture aligned with mentor feedback (high-level design, Multi-AZ, VPC endpoints, managed database).

### 3. Solution Architecture

#### Request flow
1. Users resolve the domain with **Amazon Route 53**.
2. **Amazon CloudFront** (with **AWS WAF** web ACL attached) serves the frontend from the **S3 Frontend Bucket**.
3. Dynamic actions (products, cart, orders, image upload) go from CloudFront to the **Application Load Balancer**.
4. ALB forwards traffic to **ECS Fargate** tasks in **Private Subnets**.
5. Backend APIs handle: **product**, **cart**, **order**, **user**, and **upload image**.
6. DocumentDB, secrets, and images are reached from private subnets; metrics/logs go to **CloudWatch**.

#### High-level architecture (mentor-aligned)

![SportWear High-Level Architecture](/images/2-Proposal/sportwear_architecture.png)

**Design notes (high-level)**
- Diagram focuses on major components (no detailed Security Group / OAC boxes).
- **2 AZs**, each with **Public Subnet** + **Private Subnet**.
- **VPC endpoints**: Interface/PrivateLink for **Secrets Manager**, **CloudWatch**, **ECR**; Gateway endpoint for **S3**.
- Database: **Amazon DocumentDB** (managed, MongoDB-compatible). Alternative option: **Amazon DynamoDB**.

#### AWS services used
| Layer | Service | Role |
| --- | --- | --- |
| DNS | Amazon Route 53 | Domain routing |
| Security edge | AWS WAF | Web request filtering |
| CDN | Amazon CloudFront | Cache & distribute frontend |
| Frontend storage | Amazon S3 | Static website assets |
| Load balancing | Application Load Balancer | Distribute API traffic |
| Compute | Amazon ECS Fargate | Run backend containers |
| Container registry | Amazon ECR | Store Docker images |
| Secrets | AWS Secrets Manager | DB URL, JWT secret, API keys |
| Observability | Amazon CloudWatch | Logs & metrics |
| Private access | VPC endpoints | PrivateLink (ECR, Secrets Manager, CloudWatch) + S3 Gateway |
| Database | Amazon DocumentDB | Managed MongoDB-compatible database |
| Networking | VPC (2 AZ) | Public + Private subnets for HA |

### 4. Technical Implementation

**Phases**
1. Foundation: IAM, VPC, EC2/S3 basics, CLI, monitoring.
2. Architecture design: SportWear high-level design, service selection, Multi-AZ VPC.
3. Deploy frontend: S3 + CloudFront.
4. Deploy backend: Docker → ECR → ECS Fargate + ALB.
5. Secure & operate: WAF, Secrets Manager, CloudWatch, DB connectivity, API tests.
6. Optimize: mentor feedback, Multi-AZ, VPC endpoints, HA checklist & docs.

**Core APIs**
- Product, Cart, Order, User, Upload image

### 5. Timeline & Milestones

| Period | Focus |
| --- | --- |
| Weeks 1–6 | AWS fundamentals (Console, EC2, S3, CloudFront, CloudWatch, CLI) |
| Week 7 | SportWear requirements & high-level architecture |
| Week 8 | Frontend (S3/CloudFront) + Backend (ECS/ECR/ALB) |
| Week 9 | WAF, Secrets Manager, CloudWatch, DB & API testing |
| Week 10 | Multi-AZ & VPC endpoint optimization |
| Week 11 | Production-style architecture, e-commerce tests, docs & finalize report (24/07 – 31/07) |

### 6. Budget Estimation

Target: keep lab spend low — use **Free Tier / student credits** for eligible services, and watch paid services carefully (especially DocumentDB).

Main cost drivers to watch:
- CloudFront data transfer
- ECS Fargate task hours
- DocumentDB instance hours (usually outside Free Tier)
- S3 storage & requests
- ALB hours

Use [AWS Pricing Calculator](https://calculator.aws/) before scaling up and enable billing alerts.

### 7. Risk Assessment

| Risk | Impact | Mitigation |
| --- | --- | --- |
| Unexpected AWS cost | Medium | Prefer Free Tier where possible, billing alarms, cleanup unused resources |
| AZ / service outage | Medium | Multi-AZ VPC + ALB across AZs |
| Secret leakage | High | Secrets Manager + least-privilege IAM |
| API / DB failure | Medium | CloudWatch logs/metrics + health checks |
| Architecture review changes | Low | Keep high-level diagram; iterate from mentor feedback |

### 8. Expected Outcomes

- A working SportWear website on AWS: browse, cart, order, image upload.
- Clear high-level architecture with Multi-AZ and VPC endpoints.
- Backend isolated in private subnets; frontend accelerated by CloudFront.
- Managed database (DocumentDB) and centralized secrets/monitoring.
- Internship report artifacts: proposal, worklog, workshop documentation.
