---
title: "Create an Amazon ECR Repository"
date: 2026-07-19
weight: 1
chapter: false
pre: " <b> 5.4.1. </b> "
---


## Objectives

Create a private repository to store the BravelSport backend Docker image.

## Implementation steps

1. Open Amazon ECR in the selected Region.
2. Choose **Create repository**.
3. Visibility: **Private**.
4. Suggested repository name: `web-ban-ao-backend`.
5. Enable **Image tag immutability** in `Immutable` mode to prevent existing Docker images from being overwritten. Use a unique tag for every deployment, such as `v1.0.0`, `build-001`, or a Git commit SHA.

6. Enable **Basic scanning – Scan on push** so that Amazon ECR automatically scans operating-system vulnerabilities whenever a new Docker image is pushed to the repository.
7. Create a lifecycle policy to remove old images if the workshop runs for an extended period.



![ECR repository](/images/5-Workshop/5.4-Backend-deployment/create-ecr.png)

## Verification

- The repository exists in the correct Region.
- The repository is private.
- The EC2 IAM Role has the minimum permissions required to push images.

## Common issues

| Issue | Resolution |
|---|---|
| A repository with the same name already exists | Use the existing repository after confirming that it belongs to the correct project |
| AccessDenied | Add the minimum required ECR permissions to the EC2 instance profile |
| Wrong Region | Authenticate to and push to the registry in the correct Region |

## Summary

ECR is ready to receive the backend Docker image.



- Previous section: [5.4. Backend deployment](../)
- Next section: [5.4.2. Build and push the image](../5.4.2-Build-and-push-image/)
