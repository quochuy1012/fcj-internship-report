---
title: "Create an ECS Service"
date: 2026-07-19
weight: 6
chapter: false
pre: " <b> 5.4.6. </b> "
---




## Objectives

Run the backend in Private Subnets without a public IP address and register the task in an `ip`-type Target Group.

## Implementation steps

### 1. Create the ECS Service

In the Amazon ECS Console, go to:

```text
Amazon ECS → Clusters → web-ban-ao-cluster
```

Choose **Create service**.

Configure the Service as follows:

| Parameter | Value |
|---|---|
| Cluster | `web-ban-ao-cluster` |
| Launch type | AWS Fargate |
| Task Definition | `web-ban-ao-backend-task` |
| Revision | Latest revision |
| Service name | `web-ban-ao-backend-service` |
| Desired tasks | `1` |
| Deployment type | Rolling update |

### 2. Configure networking

Configure networking for the ECS Service:

- VPC: `bravelsport-vpc`
- Subnets: `bravelsport-private-a` and `bravelsport-private-b`
- Security Group: `SG-ECS`
- Auto-assign public IP: **Disabled**

The ECS task runs entirely in Private Subnets and accesses the Internet through the NAT Gateway.

### 3. Configure the Load Balancer

Select:

- Load balancer type: **Application Load Balancer**
- Application Load Balancer: the previously created ALB
- Container: `backend-container`
- Container port: `3000`
- Target Group: `bravelsport-backend-tg`
- Target type: `IP`

Health Check Grace Period:

```text
60 seconds
```

### 4. Create the Service

After completing the configuration, choose **Create**.

Amazon ECS will:

- Start a task from the Task Definition.
- Register the task in the Target Group.
- Perform the Health Check.
- Move the Service to **Active** when the backend is operating normally.

<!-- INSERT IMAGE HERE:
Capture the ECS Service list.
The screenshot should show:
- Cluster web-ban-ao-cluster
- Service web-ban-ao-backend-service
- Status Active
- Launch Type Fargate
- Desired Tasks = 1
- Deployment Completed

Mask the Account ID and ARN if displayed.
-->

![ECS Service](/images/5-Workshop/5.4-Backend-deployment/create-ecs-service.png)

<!-- INSERT IMAGE HERE:
Capture the Service details page.
The screenshot should show:
- Service Overview
- Status Active
- Tasks (1 Desired)
- Deployment Success
- Task Definition Revision
- Running Task

Mask the Task ID, Account ID, and ARN if necessary.
-->

![ECS Service Overview](/images/5-Workshop/5.4-Backend-deployment/ecs-service-overview.png)

## Verification

After completing the configuration, confirm that:

- The ECS Service is **Active**.
- Deployment status is **Success**.
- Desired Tasks is **1**.
- Running Tasks is **1**.
- Launch Type is **AWS Fargate**.
- The Task Definition uses the latest revision.
- The ECS task is registered in the Target Group.
- The ALB Health Check succeeds.

<!-- INSERT IMAGE HERE:
Capture the service configuration showing the VPC, private subnets, public IP disabled, and Target Group. Mask ARNs, subnet IDs, and service discovery data if displayed.
Information to mask: Account ID, ARN, public IP address, token, cookie, secret, and user data if displayed.
-->

![Create the ECS Service](/images/5-Workshop/5.4-Backend-deployment/create-ecs-service.png)

### Check the task

Open the Tasks tab and confirm that the task status is `Running`.

<!-- INSERT IMAGE HERE:
Capture the Running task. Mask the task ARN, private IP, and Account ID.
Information to mask: Account ID, ARN, public IP address, token, cookie, secret, and user data if displayed.
-->

![ECS Task Running](/images/5-Workshop/5.4-Backend-deployment/ecs-task-running.jpg)

### Check the Target Group

Wait until the target status changes to `Healthy`.

<!-- INSERT IMAGE HERE:
Capture the Healthy target status. Mask the target private IP and resource ID.
Information to mask: Account ID, ARN, public IP address, token, cookie, secret, and user data if displayed.
-->

![Healthy ECS target](/images/5-Workshop/5.4-Backend-deployment/ecs-target-healthy.png)

### Check the logs

Open CloudWatch Logs and confirm that startup logs, port `3000`, and the database connection appear when configured. Do not capture secrets or the full connection string.

<!-- INSERT IMAGE HERE:
Capture the task log stream. Mask user data, JWTs, email addresses, payment payloads, and the database URI.
Information to mask: Account ID, ARN, public IP address, token, cookie, secret, and user data if displayed.
-->

![Container logs](/images/5-Workshop/5.4-Backend-deployment/container-logs.jpg)

## Verification

- The Service is stable and its running count equals its desired count.
- The task is in a Private Subnet and public IP assignment is disabled.
- The target is Healthy.
- Logs appear in CloudWatch.
- The ALB is not connected to the EC2 Build Machine.

## Common issues

| Issue | Resolution |
|---|---|
| The task stops | Review the stopped reason, execution role, image URI, and logs |
| Target is unhealthy | Check health path `/`, the Security Groups, and port `3000` |
| Atlas connection fails | Check the NAT EIP in the IP Access List and verify `MONGO_URI` |
| The Service cannot be created | Check `iam:PassRole`, subnets, and the Target Group |

## Summary

The BravelSport backend is running on ECS Fargate and its health is monitored by the ALB.


- Previous section: [5.4.5. Create the ECS Cluster and Task Definition](../5.4.5-Create-ecs-cluster/)
- Next section: [5.5. Frontend deployment](../../5.5-Frontend-deployment/)
