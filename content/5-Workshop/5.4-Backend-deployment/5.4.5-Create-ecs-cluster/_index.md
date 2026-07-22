---
title: "Create a CloudWatch Log Group, ECS Cluster, and Task Definition"
date: 2026-07-19
weight: 5
chapter: false
pre: " <b> 5.4.5. </b> "
---



## Objectives

Define the container runtime, IAM roles, port, environment variables, secrets, and logging before creating the ECS Service.

## Implementation steps

### 1. Create a CloudWatch Log Group

Amazon ECS uses CloudWatch Logs to store all backend container logs. Creating the Log Group first allows the Task Definition to write logs as soon as the container starts.

In the AWS Management Console, go to:

```text
CloudWatch → Logs → Log groups
```

Choose:

```text
Create log group
```

Enter the following values:

| Parameter | Value |
|---|---|
| Log group name | `/ecs/bravelsport-backend` |
| Log class | `Standard` |
| Retention | `30 days` |

After entering the values, choose **Create** to create the Log Group.

A 30-day retention period is appropriate for the workshop because it keeps enough logs for troubleshooting without generating excessive storage costs.

<!-- INSERT IMAGE HERE:
Capture the CloudWatch Log Group after creation.
The screenshot should show:
- Log Group Name: /ecs/bravelsport-backend
- Retention: 30 days
- Log Class: Standard

Mask the Account ID and ARN if displayed.
-->

![CloudWatch Log Group](/images/5-Workshop/5.4-Backend-deployment/cloudwatch-log-group.png)

This CloudWatch Log Group will be referenced by the Task Definition through the `awslogs` log driver, allowing all NestJS application logs to be recorded during operation.

### 2. Create the ECS Cluster

- Cluster name: `bravelsport-cluster`.
- Infrastructure: AWS Fargate.
- Container Insights: Enabled.

### 3. Create the Task Definition

- Launch type: AWS Fargate.
- Network mode: `awsvpc`.
- Task CPU: `512 CPU units (0.5 vCPU)`.
- Task memory: `1024 MiB (1 GiB)`.
- Task Execution Role: `ecsTaskExecutionRole`.
- Task Role: the IAM Role used by the backend application to access S3 Media.
- Operating system: Linux.
- CPU architecture: `X86_64`.

Container configuration:

- Container name: `backend-container`.
- Image URI: `<ECR_REPOSITORY_URI>:latest`.
- Essential: Yes.
- Container port: `3000/TCP`.
- Log driver: `awslogs`.
- Log group: `/ecs/bravelsport-backend`.
- Stream prefix: `ecs`.

<!-- INSERT IMAGE HERE:
Capture the image field with the Account ID masked, port mapping 3000, and awslogs. Do not display plain-text secrets.
Information to mask: Account ID, ARN, public IP address, token, cookie, secret, and user data if displayed.
-->

![Container configuration](/images/5-Workshop/5.4-Backend-deployment/container-configuration.jpg)

Non-sensitive environment variables can be declared directly in the Task Definition.

Secrets that must be managed include `MONGO_URI`, `JWT_SECRET`, Google, VNPay, Cloudinary, and email credentials when used.

<!-- INSERT IMAGE HERE:
Capture the Task Definition family/revision, execution role, task role, CPU/memory, and container. Mask the ARN, Account ID, and secret references if necessary.
Information to mask: Account ID, ARN, public IP address, token, cookie, secret, and user data if displayed.
-->

![ECS Task Definition](/images/5-Workshop/5.4-Backend-deployment/create-ecs-task-definition.png)

## Verification

- The Task Definition uses Fargate and `awsvpc`.
- The port mapping is `3000`.
- The Execution Role and Task Role are assigned correctly.
- `awslogs` points to the correct Log Group.
- No secrets are stored in plain text.

## Common issues

| Issue | Resolution |
|---|---|
| Invalid CPU/memory combination | Select a supported Fargate combination; retain a placeholder until the value is confirmed |
| Image architecture mismatch | Rebuild the image for the correct platform or change the Task architecture |
| Log group not found | Create the Log Group first or enable automatic creation with the required permissions |
| Secret ARN is in the wrong Region | Create the secret or parameter in the same Region, or correct the reference |

## Summary

The Cluster and Task Definition now fully describe the backend runtime.



- Previous section: [5.4.4. Create the ALB](../5.4.4-Create-alb/)
- Next section: [5.4.6. Create the ECS Service](../5.4.6-Create-ecs-service/)
