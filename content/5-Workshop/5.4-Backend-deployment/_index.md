---
title: "Deploy the NestJS backend on ECS Fargate"
date: 2026-07-19
weight: 4
chapter: false
pre: " <b> 5.4. </b> "
---


## Objectives

Build the backend on EC2, store the image in ECR, configure IAM and CloudWatch, and then run the container on ECS Fargate behind an ALB and an `ip`-type Target Group.

## Implementation steps

1. [5.4.1. Create an Amazon ECR Repository](./5.4.1-Create-ecr/)
2. [5.4.2. Build and push the Docker image](./5.4.2-Build-and-push-image/)
3. [5.4.3. Create IAM Roles](./5.4.3-Create-iam-roles/)
4. [5.4.4. Create a Target Group and ALB](./5.4.4-Create-alb/)
5. [5.4.5. Create a CloudWatch Log Group, ECS Cluster, and Task Definition](./5.4.5-Create-ecs-cluster/)
6. [5.4.6. Create an ECS Service](./5.4.6-Create-ecs-service/)


![Backend deployment overview](/images/5-Workshop/5.4-Backend-deployment/backend-deployment-overview.png)

## Principles

- EC2 does not run the production backend.
- The Target Group must use the `ip` target type for Fargate with `awsvpc` networking.
- The container port is `3000`.
- The initial health check path is `/` and must be confirmed using an actual response.
- Secret mechanism: `AWS Systems Manager Parameter Store (SecureString)`.
- Do not store the MongoDB URI or JWT secret in IAM, source code, or the Docker image.

## Verification

- ECR contains the selected image digest and tag.
- The Task Definition uses the correct image URI, execution role, task role, and log configuration.
- The ECS task is in a Private Subnet and public IP assignment is disabled.
- The Target Group displays a Healthy status.
- CloudWatch Logs receives stdout and stderr output.

## Common issues

| Issue | Resolution |
|---|---|
| `CannotPullContainerError` | Check the ECR URI, execution role, NAT route, and image architecture |
| Target is unhealthy | Check `/`, port 3000, the Security Groups, and the bind address |
| Task stops immediately | Open the stopped reason and CloudWatch log |
| A secret is exposed | Rotate the secret and move it to SSM or Secrets Manager |

## Summary

The backend follows an immutable-container deployment model: it is built on EC2, stored in ECR, and run on ECS Fargate.



- Previous section: [5.3. Building the network infrastructure](../5.3-Network-infrastructure/)
- Next section: [5.4.1. Create ECR](./5.4.1-Create-ecr/)
