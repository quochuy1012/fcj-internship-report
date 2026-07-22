---
title: "Create IAM Roles for ECS"
date: 2026-07-19
weight: 3
chapter: false
pre: " <b> 5.4.3. </b> "
---



## Objectives

Separate permissions used by the ECS platform from permissions used by the application code.

## Implementation steps
### 1. ECS Task Execution Role

Trusted entity: `ecs-tasks.amazonaws.com`.

Primary permissions:

- Pull Docker images from Amazon ECR.
- Create log streams and send container logs to Amazon CloudWatch Logs.
- Read secrets and configuration from AWS Secrets Manager when referenced by the ECS Task Definition.

Attach the following managed policy to the role:

- `AmazonECSTaskExecutionRolePolicy`

Also attach the following policy:

- `SecretsManagerAccess`

This policy allows ECS to retrieve secrets such as `MONGO_URI`, `JWT_SECRET`, and other sensitive values from AWS Secrets Manager when the task starts.

### 2. ECS Task Role

Trusted entity: `ecs-tasks.amazonaws.com`.

Application permissions:

- `s3:GetObject`, `s3:PutObject`, and `s3:DeleteObject` for objects in the BravelSport S3 Media Bucket.
- `s3:ListBucket` for the S3 Media Bucket when the application needs to list objects.
- Add other AWS API permissions only when the source code actually uses them.

Permissions must be limited to the exact ARN of the S3 Media Bucket and must not apply to every S3 resource in the account.

Do not attach `AdministratorAccess` or `AmazonS3FullAccess`.

### 3. Secrets and IAM

IAM does not store:

- `MONGO_URI`.
- `JWT_SECRET`.
- Google client secret.
- VNPay hash secret.
- Cloudinary API secret.
- SMTP password.

Secret storage mechanism: `[TO BE CONFIRMED: SSM Parameter Store or AWS Secrets Manager]`.

<!-- INSERT IMAGE HERE:
Capture the list or detail pages for both roles. Mask full ARNs and the Account ID; do not open a tab that displays secret values.
Information to mask: Account ID, ARN, public IP address, token, cookie, secret, and user data if displayed.
-->

![Create ECS IAM roles - step 1](/images/5-Workshop/5.4-Backend-deployment/create-iam-roles1.png)

![Create ECS IAM roles - step 2](/images/5-Workshop/5.4-Backend-deployment/create-iam-roles2.png)

## Verification

- The Task Execution Role and Task Role are separate roles.
- The trust policy uses `ecs-tasks.amazonaws.com`.
- The Task Role contains only the S3 Media permissions required by the application.
- No secrets are included in an IAM policy.

## Common issues

| Issue | Resolution |
|---|---|
| The application cannot access S3 | Attach the policy to the Task Role, not the Execution Role |
| ECS cannot pull the image | Check the Execution Role and ECR permissions |
| The service creator cannot pass the role | Grant restricted `iam:PassRole` permission for the deployment roles |
| IAM is used to “store a secret” | Move the value to SSM or Secrets Manager |

## Summary

The two roles are separated according to the responsibilities of the ECS agent and the application.



- Previous section: [5.4.2. Build and push the image](../5.4.2-Build-and-push-image/)
- Next section: [5.4.4. Create the ALB and Target Group](../5.4.4-Create-alb/)
