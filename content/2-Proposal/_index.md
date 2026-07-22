---
title: "Proposal"
date: 2026-07-19
weight: 2
chapter: false
pre: " <b> 2. </b> "
---

# BravelSport Sports Web Platform

## AWS architecture proposal for a secure, scalable deployment

### 1. Executive Summary

**BravelSport** is a sports web platform available at [bravelsport.com](https://bravelsport.com/). It serves customers, players, clubs, court owners, and administrators.

Core features include product browsing/search, accounts, cart and orders, court booking and schedule management, image uploads, and admin management of users, products, orders, and courts.

This proposal describes an AWS deployment that separates frontend, backend, networking, storage, monitoring, and database.

- Frontend: static build on **Amazon S3**, delivered by **Amazon CloudFront**
- Domain: **Amazon Route 53** for `bravelsport.com`
- Edge protection: **AWS WAF** associated with CloudFront
- Backend: Docker images in **Amazon ECR**, run on **Amazon ECS Fargate** in private subnets
- Ingress: **Application Load Balancer** forwards API traffic to ECS
- Database: **MongoDB Atlas** (external to AWS)
- Media/config: **Amazon S3**; access via **AWS IAM**; telemetry via **Amazon CloudWatch**

The design fits workshop / MVP / small–medium workloads.

### 2. Problem Statement

Without a clear architecture, BravelSport risks:

- Backend exposed directly to the Internet
- Frontend and backend coupled on one host
- Media lost when containers are replaced
- Hard backend versioning/rollback
- No central logs
- Poor scale characteristics
- Secrets leaked from source code
- AWS cost continuing after the workshop

**Solution:** separate frontend (S3 + CloudFront), backend (private ECS Fargate), media (S3), and data (MongoDB Atlas).

### 3. Deployment Goals

- Clear AWS architecture for BravelSport
- Separate frontend / backend / media / database
- No public IP on ECS tasks
- Docker + ECR for backend packaging
- S3 for media; CloudWatch for logs/metrics; IAM for access
- Scale by increasing ECS tasks
- Track cost and clean up unused resources

### 4. Solution Architecture

Three zones: **Edge & Frontend**, **VPC & Backend**, **Supporting services & storage**.

![BravelSport AWS Architecture](/images/2-Proposal/bravelsport_aws_architecture.png)

#### 4.1 Edge & Frontend

Route 53 resolves `bravelsport.com` to CloudFront. WAF inspects requests on the distribution. CloudFront serves the private S3 Frontend Bucket (no direct public bucket access).

#### 4.2 VPC & Backend

ALB and NAT Gateway sit in public subnets; ECS Fargate tasks sit in private subnets without public IPs. The ALB forwards API traffic to ECS via a target group and health checks. ECS inbound is limited to the ALB security group.

Outbound from ECS to ECR, CloudWatch, S3, and MongoDB Atlas goes through NAT Gateway → Internet Gateway.

#### 4.3 Supporting services & storage

- **ECR** — backend images
- **CloudWatch** — logs/metrics
- **S3 Frontend / Media / (optional) private config** buckets
- **IAM** — least-privilege roles
- **MongoDB Atlas** — application data

Do not store credentials in source code or images. If a private config bucket is used: Block Public Access, encryption, Task Role-only read, no public URL, not shared with the frontend bucket, never commit secrets to GitHub.

DB path: `ECS Fargate → NAT Gateway → Internet Gateway → MongoDB Atlas`  
Prefer Atlas allowlisting by NAT Elastic IP (not `0.0.0.0/0`).

### 5. AWS Services Used

| Service | Role |
|---|---|
| Route 53 | Domain / DNS to CloudFront |
| CloudFront | Frontend CDN + API path to ALB |
| AWS WAF | Edge request filtering |
| S3 | Frontend, media, optional config |
| VPC (public/private subnets) | Network isolation |
| ALB / Target Group | API ingress + health checks |
| NAT / IGW / Elastic IP | Private outbound path |
| ECS Fargate | Backend runtime |
| ECR | Container registry |
| IAM | Access control |
| CloudWatch | Observability |
| MongoDB Atlas | External database |

### 6. Main Request Flow

1. User opens `bravelsport.com`
2. Route 53 → CloudFront
3. WAF inspects the request
4. CloudFront serves static assets from S3
5. API paths go CloudFront → ALB → ECS Fargate
6. ECS runs business logic with IAM permissions
7. ECS pulls images from ECR; reads/writes media on S3
8. ECS sends logs/metrics to CloudWatch
9. ECS reaches MongoDB Atlas via NAT
10. Response returns through ALB and CloudFront

### 7. Security Design

**SG-ALB:** allow required inbound; outbound to ECS backend port only.  
**SG-ECS:** inbound only from SG-ALB; no public backend port; outbound as needed.

Also: no ECS public IPs; S3 Block Public Access; least-privilege IAM; encrypt credentials; Atlas restricted to NAT EIP; test WAF in Count before Block.

### 8. Three-Month Roadmap

| Month | Focus | Outcome |
|---|---|---|
| 1 | Learn AWS services + draft BravelSport architecture | Service notes + initial diagram |
| 2 | Docker/ECR/ECS trials, S3/CloudFront, VPC/ALB, Atlas connectivity | Working component proofs |
| 3 | Full deploy, integrate S3/Atlas/IAM/CloudWatch, test + docs | Live workshop system + guide |

### 9. Cost & Optimization

Main drivers: Route 53, CloudFront, S3, WAF, ALB, ECS Fargate, NAT Gateway, Elastic IP, ECR, CloudWatch, MongoDB Atlas.

Estimate with the [AWS Pricing Calculator](https://calculator.aws/) for the chosen Region and measured usage.

Optimize by right-sizing tasks, shortening lab runtime, deleting unused NAT/ALB/EIP/images, limiting log retention, tuning CloudFront cache, and cleaning up after the workshop.

### 10. Risks & Mitigations

| Risk | Mitigation |
|---|---|
| Public S3 | Block Public Access + policy review |
| Leaked credentials | Encrypt + Task Role only; never commit secrets |
| ECS cannot pull image | Check ECR, IAM, NAT |
| ALB health check fails | Match port/path/security groups |
| Atlas unreachable | Check EIP, NAT route, credentials |
| WAF false positives | Count mode first |
| NAT cost growth | Monitor and clean up |

### 11. Expected Outcomes

BravelSport on AWS with Route 53 domain, S3+CloudFront frontend, private ECS Fargate backend behind ALB, ECR images, S3 media, MongoDB Atlas data, CloudWatch telemetry, IAM controls, plus clear test and cleanup procedures.

### 12. Future Improvements

CloudWatch Alarms, AWS Budgets, CI/CD, ECS Auto Scaling, multi-AZ, VPC Endpoints to reduce NAT traffic, dedicated secrets management, and backups for S3/Atlas.

### 13. Conclusion

Separating frontend, backend, media, and database gives BravelSport a clearer AWS deployment path suitable for the workshop and ready to grow later.
