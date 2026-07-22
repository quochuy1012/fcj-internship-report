---
title: "Database and media"
date: 2026-07-19
weight: 6
chapter: false
pre: " <b> 5.6. </b> "
---



## Objectives

Configure MongoDB Atlas outside the AWS VPC and migrate the media mechanism from Cloudinary or local `/uploads` storage to S3 Media within the scope of the workshop.

## Data architecture

```text
ECS Fargate → Private Route Table → NAT Gateway/EIP → IGW → MongoDB Atlas
ECS Fargate --Task Role--> S3 Media Bucket
```

<!-- INSERT IMAGE HERE:
Draw MongoDB Atlas outside the VPC, the NAT EIP in the Atlas Access List, and S3 Media accessed through the ECS Task Role.
Information to mask: Account ID, ARN, public IP address, token, cookie, secret, and user data if displayed.
-->

![Data and media overview](/images/5-Workshop/5.6-Data-and-media/data-and-media-overview.png)

## Implementation steps

1. [5.6.1. Configure MongoDB Atlas](./5.6.1-Configure-mongodb-atlas/)
2. [5.6.2. Create an S3 Media Bucket and update the backend](./5.6.2-Create-media-bucket/)

### Notes

The current source code creates a local `uploads` directory and supports Cloudinary. Therefore, the documentation must not state that the application already has a complete S3 integration. The S3 section requires code changes, environment variables, IAM permissions, and testing.

## Verification

- The Atlas IP Access List contains only the NAT EIP and necessary management IP addresses.
- `0.0.0.0/0` is not used in the final configuration.
- `MONGO_URI` is injected through a secret mechanism that has not yet been confirmed.
- Uploaded media actually appears in S3 before the task is marked complete.

## Common issues

| Issue | Resolution |
|---|---|
| Atlas connection times out | Check the NAT route, EIP access list, and Security Group egress |
| Media is lost after a task restart | Complete the migration from local uploads to S3 or continue using Cloudinary |
| S3 returns AccessDenied | Check the Task Role, bucket policy, and object key |

## Summary

The database and media have been separated from the container's temporary filesystem.



- Previous section: [5.5. Frontend deployment](../5.5-Frontend-deployment/)
- Next section: [5.6.1. Configure MongoDB Atlas](./5.6.1-Configure-mongodb-atlas/)
