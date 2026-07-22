---
title: "Prerequisites"
date: 2026-07-19
weight: 2
chapter: false
pre: " <b> 5.2. </b> "
---



## Objectives

Prepare the account, IAM identity, EC2 Ubuntu Build Machine, tools, source code, MongoDB Atlas, DNS, certificate, and deployment parameter table before creating AWS resources.

## Implementation steps

### 1. Prepare the account and IAM identity

- Do not use the root user for day-to-day operations.
- Create or use an IAM identity with permissions appropriate for the workshop.
- Enable MFA whenever possible.
- Attach an IAM Role to EC2 through an Instance Profile instead of storing long-lived access keys.
- Do not grant `AdministratorAccess` when the team can create a more restricted policy.

Verify the identity from EC2:

```bash
aws sts get-caller-identity
```


### 2. Select an AWS Region

Select the Region for the VPC, EC2, ECR, ECS, ALB, S3 Media, and CloudWatch: `[TO BE CONFIRMED]`.

The certificate used by CloudFront must be created or imported in `us-east-1`. Do not insert a different Region into the documentation until the team has confirmed it.

### 3. Create an EC2 Ubuntu Build Machine

Create an EC2 Ubuntu instance in the Development & Deployment environment.

Requirements:

- Instance type: `[TO BE CONFIRMED]`.
- Storage: `[TO BE CONFIRMED]`.
- IAM instance profile: minimum permissions required to push to ECR, upload to S3, and run the necessary deployment commands.
- Do not open port `3000`.
- When using SSH, open port `22` only from the management IP `[TO BE CONFIRMED]`.
- Prefer Systems Manager Session Manager when the account and AMI have been configured appropriately.


![Running EC2 Build Machine](/images/5-Workshop/5.2-Prerequisite/ec2-running.jpg)

### 4. Install tools

On Ubuntu:

```bash
sudo apt update
sudo apt install -y git unzip ca-certificates curl

aws --version
docker --version
node --version
npm --version
git --version
```

Install the Node.js version required by `package.json` and confirm that the Docker daemon is running.

![Verify local tools](/images/5-Workshop/5.2-Prerequisite/verify-local-tools.jpg)

### 5. Clone the source code

```bash
git clone https://github.com/huu-luan-2004/ShopBanDoAo.git
cd ShopBanDoAo
```

Record the commit used for the workshop:

```bash
git rev-parse --short HEAD
```


### 6. Test the frontend build

```bash
cd Fronted
npm ci
npm run build
ls -la dist
```

Do not place secrets in variables prefixed with `VITE_`, because these variables may be embedded in the static bundle.

<!-- INSERT IMAGE HERE:
Capture the terminal after npm run build completes and show the file list in dist. Mask internal URLs or tokens if they appear.
Information to mask: Account ID, ARN, public IP address, token, cookie, secret, and user data if displayed.
-->

<!-- Image not yet available under static: /images/5-Workshop/5.2-Prerequisite/frontend-build-success.png -->

### 7. Test the backend Docker image build

From the directory containing the appropriate Dockerfile:

```bash
cd ../backend-nestjs
docker build -t bravelsport-backend:local .
docker image ls bravelsport-backend
```

Run the container with placeholder environment variables or a safe test environment:

```bash
docker run --rm -p 3000:3000 \
  --env-file <SAFE_ENV_FILE> \
  bravelsport-backend:local
```

Verify it:

```bash
curl -i http://localhost:3000/
```

The initial health endpoint is `/`. If the application returns a status code other than `200`, confirm the actual route before creating the Target Group.


![Successful Docker image build](/images/5-Workshop/5.2-Prerequisite/backend-docker-build-success.jpg)

### 8. Prepare MongoDB Atlas

- Create a cluster.
- Create a dedicated database user for the application with minimum required permissions.
- Do not use `0.0.0.0/0` in the final configuration.
- After creating the NAT Gateway, add its Elastic IP to the IP Access List.
- Do not include the connection string in screenshots or documentation.



![MongoDB Atlas Network Access](/images/5-Workshop/5.2-Prerequisite/mongodb-network-access.jpg)

### 9. Prepare the Route 53 Hosted Zone

Confirm the Hosted Zone for the `bravelsport.com` domain. Do not modify records currently serving production until a migration plan is available.


![Route 53 Hosted Zone](/images/5-Workshop/5.2-Prerequisite/route53-hosted-zone.jpg)

### 10. Prepare the ACM certificate

In the `us-east-1` Region, request a certificate for the hostname used with CloudFront. Complete DNS validation and wait until its status becomes `Issued`.

![ACM certificate Issued](/images/5-Workshop/5.2-Prerequisite/acm-certificate-issued.jpg)


## Verification

- `aws sts get-caller-identity` runs successfully.
- The Docker daemon is running.
- `npm run build` creates the `dist/` directory.
- The backend Docker image runs and listens on port `3000`.
- The health path `/` has been tested or marked for confirmation.
- The Hosted Zone and ACM certificate are ready.
- No secrets appear in the terminal, screenshots, or Git history.

## Common issues

| Issue | Cause | Resolution |
|---|---|---|
| AWS CLI returns AccessDenied | The IAM identity lacks permissions | Add the minimum required permissions or verify the instance profile |
| Docker permission denied | The user is not in the docker group | Temporarily use `sudo` or configure the docker group according to the proper procedure |
| Frontend build fails | Incompatible Node/npm version or missing environment variables | Check the lock file and `.env.example` |
| Container does not expose port 3000 | Incorrect Dockerfile or `PORT` variable | Check logs and port mapping |
| ACM certificate is not Issued | DNS validation is incomplete | Check the validation CNAME in Route 53 |

## Summary

The build environment, source code, domain, certificate, and deployment values have been prepared. EC2 is used only for builds and deployment and does not participate in runtime request handling.



- Previous section: [5.1. Workshop overview](../5.1-Workshop-overview/)
- Next section: [5.3. Building the network infrastructure](../5.3-Network-infrastructure/)
