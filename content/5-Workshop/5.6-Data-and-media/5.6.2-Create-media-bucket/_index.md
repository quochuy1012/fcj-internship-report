---
title: "Create an S3 Media Bucket and update the backend"
date: 2026-07-19
weight: 2
chapter: false
pre: " <b> 5.6.2. </b> "
---


## Objectives

Replace media storage on the local filesystem or Cloudinary with Amazon S3 to ensure durable data storage when deploying on Amazon ECS Fargate.

## Current source-code behavior

The backend currently creates an `uploads` directory and serves the `/uploads/` URL; the sample configuration file also supports Cloudinary. The local filesystem of a Fargate task is not suitable for durable media storage across deployments. Therefore, the workshop uses Amazon S3 as Media Storage.

## Implementation steps

### 1. Create an S3 Media Bucket

Create a bucket with the following settings:

| Parameter | Value |
|---|---|
| Bucket name | `bravel-uploads` |
| Region | `ap-southeast-1 (Singapore)` |
| Block Public Access | Enabled |
| Default Encryption | Enabled (SSE-S3) |
| Versioning | Enabled (recommended) |
| Lifecycle | Configure according to storage requirements |

This bucket is used only for application media and is separate from the S3 Frontend Bucket.

<!-- INSERT IMAGE HERE:
Capture the bucket list or the bravel-uploads bucket page.
The screenshot should show:
- Bucket name: bravel-uploads
- Region: ap-southeast-1

Mask the Account ID if displayed.
-->

![Create the S3 media bucket](/images/5-Workshop/5.6-Data-and-media/create-media-bucket.jpg)

### 2. Grant S3 access

Create the following IAM Policy:

```text
BackendUploadS3Policy
```

This policy grants the ECS Task Role the following permissions:

- `s3:GetObject`
- `s3:PutObject`
- `s3:DeleteObject`
- `s3:ListBucket`

Limit the permissions to this bucket only:

```text
bravel-uploads
```

Do not use `AmazonS3FullAccess` or `AdministratorAccess`.

### 3. Attach permissions to the Task Role

Restrict the policy to the required bucket and media prefix.

<!-- INSERT IMAGE HERE:
Capture the policy showing the exact bucket and prefix. Mask the ARN and Account ID, and do not display unrelated policies.
Information to mask: Account ID, ARN, public IP address, token, cookie, secret, and user data if displayed.
-->

![Media Task Role policy](/images/5-Workshop/5.6-Data-and-media/media-task-role-policy.jpg)

### 4. Deploy a new revision

- Build a new image on EC2.
- Push it to ECR.
- Create a new Task Definition revision.
- Update the ECS Service.
- Monitor the deployment and logs.

### 5. Test media uploads

- Upload an image through the user interface or API.
- Confirm that the object appears in S3.
- Confirm that the media URL is displayed on BravelSport.
- Verify that restarting a task does not remove the media.

<!-- INSERT IMAGE HERE:
Capture the object in S3 and the image displayed on BravelSport. Mask user names, metadata, and signed URLs containing tokens.
Information to mask: Account ID, ARN, public IP address, token, cookie, secret, and user data if displayed.
-->

![Verify media upload](/images/5-Workshop/5.6-Data-and-media/verify-media-upload.jpg)

## Verification

- The S3 bucket is private.
- The application uploads through the Task Role.
- No AWS access key is stored in the Task Definition.
- Media remains available after a task is replaced.

## Common issues

| Issue | Resolution |
|---|---|
| AccessDenied | Check the Task Role resource ARN and prefix |
| Images are not displayed | Verify the URL strategy, CloudFront or media delivery configuration, and CORS |
| Files are uploaded locally instead of to S3 | Check the code path and environment flag |
| The bucket is public | Keep the bucket private; use CloudFront or signed URLs when needed |

## Summary

Media has been migrated to durable object storage, or the section is clearly marked as incomplete when the code changes and real evidence are not yet available.


- Previous section: [5.6.1. MongoDB Atlas](../5.6.1-Configure-mongodb-atlas/)
- Next section: [5.7. End-to-end testing](../../5.7-Testing/)
