---
title: "Build and push the Docker image"
date: 2026-07-19
weight: 2
chapter: false
pre: " <b> 5.4.2. </b> "
---


## Objectives

Build the NestJS backend on the EC2 Ubuntu instance, validate the image, and push it to ECR.

## Implementation steps

### 1. Clone or update the source code

```bash
cd ~/ShopBanDoAo
git fetch --all
git checkout <COMMIT_OR_BRANCH>
git rev-parse --short HEAD
```


### 2. Build the Docker image

```bash
cd backend-nestjs
docker build -t bravelsport-backend:<IMAGE_TAG> .
```

Do not copy `.env` files or secrets into the image. Review `.dockerignore`.

### 3. Test the image locally

```bash
docker image inspect bravelsport-backend:<IMAGE_TAG>
docker run --rm -p 3000:3000 --env-file <SAFE_ENV_FILE> \
  bravelsport-backend:<IMAGE_TAG>
curl -i http://localhost:3000/
```

### 4. Authenticate to ECR

```bash
aws ecr get-login-password --region <AWSx1_REGION> \
| docker login --username AWS --password-stdin \
  <ACCOUNT_ID>.dkr.ecr.<AWS_REGION>.amazonaws.com
```

Do not include the real Account ID in the documentation or screenshots.

### 5. Tag and push the image

```bash
docker tag bravelsport-backend:<IMAGE_TAG> \
  <ECR_REPOSITORY_URI>:<IMAGE_TAG>

docker push <ECR_REPOSITORY_URI>:<IMAGE_TAG>
```

<!-- INSERT IMAGE HERE:
Capture the completed docker build and docker push output. Mask the Account ID, full repository URI, and authentication token.
Information to mask: Account ID, ARN, public IP address, token, cookie, secret, and user data if displayed.
-->

![Build and push the backend image](/images/5-Workshop/5.4-Backend-deployment/build-and-push-image.png)

## Verification

The Amazon ECR Repository `web-ban-ao-backend` has successfully stored the backend Docker image.

Current image information:

| Parameter | Value |
| --- | --- |
| Repository | `web-ban-ao-backend` |
| Image tag | `latest` |
| Image size | `83.38 MB` |
| Image digest | `sha256:24ba0e01ba...` |
| Status | The image was successfully pushed to ECR |
| CPU architecture | Must be verified with the Docker CLI |
| Backend port | `3000` |
| Health check path | `/` |


![Docker image in Amazon ECR](/images/5-Workshop/5.4-Backend-deployment/ecr-backend-image.png)
## Common issues

| Issue | Resolution |
|---|---|
| `no basic auth credentials` | Run the ECR login command again in the correct Region |
| Push denied | Check the EC2 IAM Role and repository policy |
| The image is too large | Use a multi-stage build and remove development dependencies |
| The container binds only to localhost | Ensure that the application listens on the correct interface inside the container |

## Summary

The backend image has been built, tested, and stored in ECR.


- Previous section: [5.4.1. Create ECR](../5.4.1-Create-ecr/)
- Next section: [5.4.3. Create IAM Roles](../5.4.3-Create-iam-roles/)
