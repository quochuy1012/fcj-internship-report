---
title: "Create a Target Group and Application Load Balancer"
date: 2026-07-19
weight: 4
chapter: false
pre: " <b> 5.4.4. </b> "
---


## Objectives

Create a Target Group and Application Load Balancer to receive requests from CloudFront and forward them to ECS Fargate, while configuring a health check for the backend.

## Implementation steps

### 1. Create the Target Group

Configure the Target Group as follows:

- Target type: `IP addresses`.
- Protocol: `HTTP`.
- Port: `3000`.
- VPC: `bravelsport-vpc`.
- Health check protocol: `HTTP`.
- Health check path: `/`.
- Success codes: `200`.
- Deregistration delay: `300 seconds`.
- Healthy threshold: `5`.
- Unhealthy threshold: `2`.
- Health check timeout: `5 seconds`.
- Health check interval: `30 seconds`.

Do not register the EC2 Build Machine in the Target Group because the backend runs on Amazon ECS Fargate.

### 2. Create the Application Load Balancer

Configure the Application Load Balancer as follows:

- Scheme: `Internet-facing`.
- IP address type: `IPv4`.
- Select two Public Subnets in two Availability Zones.
- Attach Security Group `SG-ALB`.
- Listener: `HTTPS (443)`.
- Select the ACM Certificate created in the `us-east-1` Region.
- Default action: Forward to the newly created Target Group.

### 3. Restrict direct access to the ALB

To ensure that all requests pass through CloudFront and AWS WAF, apply the following controls:

- Configure the ALB Security Group to allow traffic only from CloudFront or other approved sources.
- Configure CloudFront to send a Custom Origin Header to the ALB for request validation.
- Configure the ALB Listener Rule to forward requests only when the Custom Origin Header is valid.

Do not use the ALB as a direct Internet entry point because doing so would bypass CloudFront and AWS WAF.

<!-- INSERT IMAGE HERE:
Capture the Application Load Balancer in the Active state.
The screenshot should show the HTTPS 443 Listener and Security Group.
Mask the DNS Name, ARN, Account ID, and Certificate ARN before adding it to the report.
-->

![Create the Application Load Balancer - step 1](/images/5-Workshop/5.4-Backend-deployment/create-alb1.png)

<!-- INSERT IMAGE HERE:
Capture the IP-type Target Group.
The screenshot should show port 3000, Health Check Path "/", and Target Type IP.
Mask the ARN and Account ID if displayed.
-->

![Create the Target Group - step 2](/images/5-Workshop/5.4-Backend-deployment/create-alb2.png)

## Verification

After completing the configuration, confirm that:

- The Application Load Balancer is **Active**.
- The ALB is deployed in two Public Subnets.
- The HTTPS `443` Listener is operating correctly.
- The Target Group uses **Target type = IP**.
- The backend uses port `3000`.
- The Health Check Path is `/`.
- The Success Code is `200`.
- The EC2 Build Machine is not registered in the Target Group.
- The ECS task is registered automatically in the Target Group after the Service starts.

## Common issues

| Issue | Resolution |
|---|---|
| Target Type `Instance` was selected | Recreate the Target Group with Target Type `IP` |
| The ALB does not become Active | Check the Public Subnets, Internet Gateway, Route Table, and Security Group |
| The target remains Unhealthy | Verify that the backend listens on port `3000` and that endpoint `/` returns HTTP `200` |
| The Health Check returns 404 | Verify the Health Check Path or create a dedicated endpoint |
| The ALB is directly accessible | Configure the Security Group or Custom Origin Header so that only CloudFront requests are accepted |

## Summary

The Target Group and Application Load Balancer are configured to receive requests from CloudFront, evaluate backend health, and forward requests to ECS Fargate tasks in Private Subnets.


- Previous section: [5.4.3. Create IAM Roles](../5.4.3-Create-iam-roles/)
- Next section: [5.4.5. Create an ECS Cluster and Task Definition](../5.4.5-Create-ecs-cluster/)
