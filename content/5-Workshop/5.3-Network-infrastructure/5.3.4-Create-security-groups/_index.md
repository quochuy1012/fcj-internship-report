---
title: "Create Security Groups"
date: 2026-07-19
weight: 4
chapter: false
pre: " <b> 5.3.4. </b> "
---


## Objectives

Restrict network traffic among the Application Load Balancer, ECS Fargate, and the EC2 Build Machine so that only the required components can communicate according to the system architecture.

## Implementation steps

### 1. Create a Security Group for the Application Load Balancer

Create the following Security Group:

```text
web-ban-ao-alb-sg
```

Attach this Security Group to the Application Load Balancer.

#### Inbound Rules

| Type | Protocol | Port | Source |
|---|---|---:|---|
| HTTP | TCP | 80 | 0.0.0.0/0 |
| HTTPS | TCP | 443 | 0.0.0.0/0 |

- HTTP can be used to redirect traffic to HTTPS when configured.
- HTTPS is the primary port that accepts requests from the Internet.

#### Outbound Rules

| Type | Protocol | Port | Destination |
|---|---|---:|---|
| Custom TCP | TCP | 3000 | `bravelsport-sg-ecs` |

The Application Load Balancer forwards requests to ECS Fargate only on port `3000`.

---

### 2. Create a Security Group for ECS Fargate

Create the following Security Group:

```text
bravelsport-sg-ecs
```

Attach it to the ECS Service.

#### Inbound Rules

| Type | Protocol | Port | Source |
|---|---|---:|---|
| Custom TCP | TCP | 3000 | `web-ban-ao-alb-sg` |

Only the Application Load Balancer is allowed to access the backend.

Do not open:

```text
TCP 3000 from 0.0.0.0/0
```

ECS tasks do not have public IP addresses and do not accept direct requests from the Internet.

#### Outbound Rules

Allow ECS tasks to access:

- Amazon ECR
- Amazon CloudWatch Logs
- Amazon S3
- AWS Secrets Manager
- MongoDB Atlas

For the workshop, the default outbound rule may be retained:

| Type | Protocol | Port | Destination |
|---|---|---:|---|
| All traffic | All | All | 0.0.0.0/0 |

ECS outbound traffic follows this path:

```text
ECS Fargate
→ NAT Gateway
→ Internet Gateway
→ AWS Services / MongoDB Atlas
```

---

### 3. Create a Security Group for the EC2 Build Machine

Create the following Security Group:

```text
bravelsport-sg-build
```

The EC2 Build Machine is used only to:

- Clone the source code.
- Build the React/Vite frontend.
- Build the Docker image.
- Push the image to Amazon ECR.
- Synchronize the frontend to Amazon S3.
- Deploy the application with the AWS CLI.

The EC2 Build Machine does not process user requests.

#### Inbound Rules

| Type | Protocol | Port | Source |
|---|---|---:|---|
| SSH | TCP | 22 | `<ADMIN_PUBLIC_IP>/32` |

Allow SSH only from the management IP address.

Do not open:

- TCP 3000
- HTTP 80
- HTTPS 443

When AWS Systems Manager Session Manager is used, SSH does not need to be opened.

#### Outbound Rules

| Type | Protocol | Port | Destination |
|---|---|---:|---|
| HTTPS | TCP | 443 | 0.0.0.0/0 |

EC2 uses HTTPS to access:

- GitHub
- Amazon ECR
- Amazon S3
- AWS APIs
- npm Registry
- Docker Registry

<!-- INSERT IMAGE HERE:
Capture the Security Groups for:

- web-ban-ao-alb-sg
- bravelsport-sg-ecs
- bravelsport-sg-build

The screenshots should show:

- HTTP 80
- HTTPS 443
- TCP 3000
- SSH 22

Mask:

- AWS Account ID
- Security Group ID
- VPC ID
- Management public IP
-->

![BravelSport Security Groups](/images/5-Workshop/5.3-Network-infrastructure/create-security-groups.png)

## Verification

- The Application Load Balancer exposes only HTTP `80` and HTTPS `443`.
- ECS Fargate allows inbound TCP `3000` only from the ALB Security Group.
- ECS tasks do not have public IP addresses.
- The EC2 Build Machine allows SSH only from the management IP.
- EC2 does not expose port `3000`.
- The Application Load Balancer does not forward requests to the EC2 Build Machine.
- ECS tasks can access Amazon ECR, CloudWatch Logs, Amazon S3, AWS Secrets Manager, and MongoDB Atlas through the NAT Gateway.

## Common issues

| Issue | Resolution |
|---|---|
| Target is unhealthy because of a timeout | Check SG-ALB outbound and SG-ECS inbound on port `3000` |
| The ALB can be accessed directly | Restrict access with the CloudFront origin-facing managed prefix list or a custom origin header |
| ECS cannot connect to MongoDB Atlas | Check the SG-ECS outbound rule and the MongoDB Atlas IP Access List |
| ECS cannot pull the Docker image | Check outbound access to Amazon ECR and the NAT Gateway |
| CloudWatch Logs cannot be written | Check outbound access to CloudWatch Logs and the IAM Execution Role |
| SSH is open to the entire Internet | Restrict it to the management IP or use AWS Systems Manager Session Manager |

## Summary

The Security Groups follow the **least privilege** principle and separate access among the Application Load Balancer, ECS Fargate, and the EC2 Build Machine. The backend accepts requests only from the Application Load Balancer, while the EC2 Build Machine is used only for building and deploying the application.


- Previous section: [5.3.3. Configure routing](../5.3.3-Configure-routing/)
- Next section: [5.4. Backend deployment](../../5.4-Backend-deployment/)
