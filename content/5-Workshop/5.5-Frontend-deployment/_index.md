---
title: "Deploy the React/Vite frontend"
date: 2026-07-19
weight: 5
chapter: false
pre: " <b> 5.5. </b> "
---



## Objectives

Build the frontend on EC2, upload `dist/` to a private S3 bucket, and configure CloudFront as the unified entry point for the frontend, REST API, and Socket.IO.

## Delivery architecture

```text
User → Route 53 → CloudFront + WAF
                     ├─ /* → S3 Frontend Bucket
                     ├─ /api/* → ALB
                     └─ /socket.io/* → ALB
```

<!-- INSERT IMAGE HERE:
Draw Route 53 → CloudFront/WAF with an S3 origin and an ALB origin. Clearly label the /api/* and /socket.io/* behaviors.
Information to mask: Account ID, ARN, public IP address, token, cookie, secret, and user data if displayed.
-->

![Frontend delivery overview](/images/5-Workshop/5.5-Frontend-deployment/frontend-delivery-overview.png)

## Implementation steps

1. [5.5.1. Build the React/Vite frontend](./5.5.1-Build-frontend/)
2. [5.5.2. Create the S3 Frontend Bucket](./5.5.2-Create-s3-frontend/)
3. [5.5.3. Create the CloudFront Distribution](./5.5.3-Create-cloudfront/)
4. [5.5.4. Configure AWS WAF](./5.5.4-Configure-waf/)
5. [5.5.5. Configure Route 53](./5.5.5-Configure-route53/)

## Verification

- The S3 Frontend Bucket is private and can be read only through CloudFront OAC.
- The CloudFront default behavior serves the SPA.
- `/api/*` and `/socket.io/*` do not cache dynamic content.
- Required Authorization headers, query strings, and cookies are forwarded according to the source code.
- The Route 53 Alias points to CloudFront, not directly to the ALB or ECS.

## Common issues

| Issue | Resolution |
|---|---|
| SPA reload returns 403 or 404 | Configure a suitable custom error response or CloudFront Function fallback |
| The API is cached | Use a caching-disabled policy for `/api/*` |
| Socket.IO handshake fails | Check `/socket.io/*`, query strings, headers, methods, and timeouts |
| The S3 bucket is public | Enable Block Public Access and use OAC |

## Summary

The frontend and backend are published through a single CloudFront distribution protected by WAF.



- Previous section: [5.4. Backend deployment](../5.4-Backend-deployment/)
- Next section: [5.5.1. Build the frontend](./5.5.1-Build-frontend/)
