---
title: "Workshop overview"
date: 2026-07-19
weight: 1
chapter: false
pre: " <b> 5.1. </b> "
---



## Objectives

Introduce the project, technology stack, target architecture, and workshop scope, while clearly distinguishing the EC2 machine used for builds from ECS Fargate used to run the production backend.

## Introduction to BravelSport

BravelSport is an e-commerce and sports facility booking platform available at `https://bravelsport.com/`. The system supports product browsing, shopping carts, orders, sports facility bookings, account management, and real-time chat.

Reference source code: `https://github.com/huu-luan-2004/ShopBanDoAo`.

Main technologies:

- Frontend: React, Vite, TypeScript.
- Backend: NestJS, TypeScript.
- Database: MongoDB through Mongoose.
- Authentication: JWT.
- Realtime: Socket.IO.
- REST API path: `/api/*`.
- Socket.IO transport path: `/socket.io/*`.
- Socket.IO namespace: `/chat`.
- Default backend port: `3000`.
- Initial health check path: `/`.
- Backend packaging: Docker.



![BravelSport homepage](/images/5-Workshop/5.1-Workshop-overview/bravelsport-homepage.jpg)

## Workshop objectives

After completing the workshop, participants will be able to:

1. Design a VPC with Public and Private Subnets across two Availability Zones.
2. Use an EC2 Ubuntu instance as the build and deployment machine.
3. Build the backend Docker image, push it to ECR, and run it on ECS Fargate.
4. Deploy the React/Vite frontend to a private S3 bucket and distribute it through CloudFront OAC.
5. Route `/api/*` and `/socket.io/*` from CloudFront to the ALB.
6. Connect ECS to MongoDB Atlas through a NAT Gateway.
7. Send container logs to CloudWatch Logs.
8. Extend the media mechanism from Cloudinary or local `/uploads` storage to S3 Media.
9. Test the system, assess security, monitor costs, and delete resources.

## Three-tier architecture

| Tier | Components | Role |
|---|---|---|
| Presentation | Route 53, CloudFront, WAF, S3 Frontend | DNS, TLS, static frontend distribution, and request filtering |
| Application | ALB, `ip`-type Target Group, ECS Fargate | Run the REST API and Socket.IO on port `3000` |
| Data | MongoDB Atlas, S3 Media | Store business data and media |

## Role of the EC2 Build Machine

The EC2 Ubuntu instance is placed in the **Development & Deployment** area. It is used only to clone the source code, install tools, build the frontend, build the Docker image, authenticate to ECR, push the image, and run AWS CLI deployment commands.

Deployment flow:

```text
GitHub
  └─→ EC2 Ubuntu Build Machine
        ├─→ build frontend → S3 Frontend Bucket
        └─→ docker build → Amazon ECR → ECS Fargate pull image
```

EC2 does not receive user requests, does not connect to the ALB, and does not run the production backend.



![Build and deployment flow](/images/5-Workshop/5.1-Workshop-overview/deployment-flow.jpg)

## Role of ECS Fargate

ECS Fargate is the runtime environment for the NestJS backend. Each task runs a container on port `3000`, is placed in a Private Subnet, has no public IP address, and accepts inbound traffic only from SG-ALB through an `ip`-type Target Group.

The Task Execution Role is used by the ECS platform to pull images and send logs. The Task Role is used by application code to access S3 Media or other authorized AWS services.

## Overall architecture

The architecture is divided into the following logical areas:

- **Edge & Frontend:** Route 53, CloudFront, AWS WAF, and S3 Frontend.
- **Development & Deployment:** GitHub, EC2 Ubuntu Build Machine, and ECR.
- **Amazon VPC – Backend Network:** ALB in Public Subnets; ECS Fargate in Private Subnets; NAT Gateway, Elastic IP, and Internet Gateway.
- **Supporting Services:** IAM, CloudWatch Logs, and S3 Media.
- **External Database:** MongoDB Atlas outside the AWS VPC.



![BravelSport AWS Architecture](/images/5-Workshop/architecture/bravelsport-aws-architecture.png)

## Operational flows

### Frontend

`User → Route 53 → CloudFront + AWS WAF → S3 Frontend Bucket`

### REST API

`User → Route 53 → CloudFront → /api/* → ALB → Target Group (ip) → ECS Fargate:3000`

### Socket.IO

`User → CloudFront → /socket.io/* → ALB → ECS Fargate → namespace /chat`

`/socket.io/*` is the transport path used for the handshake and frame exchange; `/chat` is the logical namespace used by the application.

### MongoDB Atlas

`ECS Fargate → Private Route Table → NAT Gateway → Internet Gateway → MongoDB Atlas`

The Elastic IP of the NAT Gateway is added to the Atlas IP Access List. Do not use `0.0.0.0/0` in the final configuration.

### Media

`User → CloudFront → ALB → ECS Fargate → S3 Media Bucket`

The current source code may use Cloudinary or local `/uploads`; S3 Media is a migration or extension implemented during the workshop.

### Build and deployment

`GitHub → EC2 Build Machine → ECR → ECS Fargate`

Frontend branch: `EC2 Build Machine → S3 Frontend Bucket`.

## Implementation steps

1. Prepare the account, EC2 build machine, domain, certificate, and source code.
2. Build the VPC, subnets, routes, and security groups.
3. Create ECR, IAM roles, a CloudWatch Log Group, an ALB, a Target Group, and an ECS Service.
4. Build the frontend, upload it to S3, and create CloudFront, WAF, and a Route 53 Alias.
5. Configure MongoDB Atlas and extend media storage to S3.
6. Perform end-to-end tests, review security, and clean up resources.

## Verification

- The diagram does not contain an `ALB → EC2` path.
- Users do not directly access EC2, the NAT Gateway, the Internet Gateway, or ECS.
- CloudFront has separate behaviors for `/api/*` and `/socket.io/*`.
- The Target Group uses the `ip` target type.
- MongoDB Atlas is located outside the VPC.
- IAM is shown with dashed association lines rather than data-flow lines.

## Common issues

| Issue | Resolution |
|---|---|
| EC2 is drawn between the ALB and ECS | Move EC2 to the Development & Deployment area |
| `/socket.io/*` and `/chat` are merged | Clearly state that the transport path and namespace are different concepts |
| S3 Media is presented as already implemented | State that it is a workshop extension |
| CloudWatch and ECR are grouped behind a generic “public endpoints” node | Connect ECS directly to each service or clearly label the path as an association or outbound connection |

## Summary

The workshop uses EC2 for builds, ECR for image storage, and ECS Fargate for the backend runtime. Runtime traffic passes through CloudFront, WAF, and the ALB; the database remains outside the VPC; and media storage is gradually migrated to S3.

## Image inventory

| No. | File name | Image content | Placement |
|---:|---|---|---|
| 1 | `bravelsport-homepage.jpg` | BravelSport homepage accessed through the project domain | After the project introduction |
| 2 | `deployment-flow.jpg` | GitHub → EC2 Build Machine → ECR → ECS Fargate flow | After the EC2 Build Machine role section |
| 3 | `bravelsport-aws-architecture.png` | Revised overall AWS architecture diagram | After the overall architecture section |

## Navigation

- Previous section: [Workshop](../)
- Next section: [5.2. Prerequisites](../5.2-Prerequisite/)
