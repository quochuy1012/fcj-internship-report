---
title: "Create a CloudFront Distribution"
date: 2026-07-19
weight: 3
chapter: false
pre: " <b> 5.5.3. </b> "
---



## Objectives

Configure Amazon CloudFront to use the private S3 bucket as the frontend origin and the Application Load Balancer as the origin for the REST API and Socket.IO.

## Implementation steps

### 1. Create the S3 Origin and Origin Access Control (OAC)

1. Create a CloudFront Distribution.
2. Select bucket `myapp-frontend-sport` as the S3 Origin.
3. Create an **Origin Access Control (OAC)** using **Sign requests (recommended)**.
4. Update the Bucket Policy so that only the CloudFront Distribution can read objects from the bucket.

The Bucket Policy has the following structure:

```json
{
  "Version": "2008-10-17",
  "Statement": [
    {
      "Sid": "AllowCloudFrontServicePrincipal",
      "Effect": "Allow",
      "Principal": {
        "Service": "cloudfront.amazonaws.com"
      },
      "Action": "s3:GetObject",
      "Resource": "arn:aws:s3:::myapp-frontend-sport/*",
      "Condition": {
        "StringEquals": {
          "AWS:SourceArn": "arn:aws:cloudfront::<ACCOUNT_ID>:distribution/<DISTRIBUTION_ID>"
        }
      }
    }
  ]
}
```

<!-- INSERT IMAGE HERE:
Capture the Bucket Policy after creating OAC.
Mask the Account ID and Distribution ID.
-->

![CloudFront OAC Bucket Policy](/images/5-Workshop/5.5-Frontend-deployment/cloudfront-oac-policy.png)

### 2. Create the ALB Origin

Add a new CloudFront Origin with these settings:

- Origin domain: the DNS Name of the Application Load Balancer.
- Origin protocol policy: **HTTP Only**.
- Origin Shield: Disabled.
- Do not configure an Origin Path.
- Optionally add a Custom Header to restrict direct access to the ALB.

### 3. Default Behavior `/*`

Configure:

- Origin: `myapp-frontend-sport`
- Viewer protocol policy: **Redirect HTTP to HTTPS**
- Allowed methods: `GET`, `HEAD`
- Compress objects automatically: Enabled
- Cache policy: **CachingOptimized**

This behavior serves all frontend content, including:

- `index.html`
- CSS
- JavaScript
- Images
- Fonts

### 4. Behavior `/api/*`

Configure:

- Origin: Application Load Balancer
- Allowed methods:

```text
GET
HEAD
OPTIONS
PUT
POST
PATCH
DELETE
```

- Cache policy: **CachingDisabled**
- Origin request policy: **AllViewerExceptHostHeader**
- Forward:
  - Authorization
  - Content-Type
  - Query Strings
  - Cookies

This behavior forwards all REST API requests to the NestJS backend.

### 5. Behavior `/socket.io/*`

Configure:

- Origin: Application Load Balancer
- Cache policy: **CachingDisabled**
- Allowed methods:

```text
GET
HEAD
OPTIONS
POST
```

- Forward all Query Strings.
- Forward the headers required for WebSocket Upgrade.
- Do not create a `/chat` Behavior, because `/chat` is a Socket.IO Namespace rather than a URL Path.

### 6. React Router fallback

In the CloudFront Distribution, configure Custom Error Responses:

| HTTP Error | Response Page | HTTP Response Code |
|---|---|---|
| 403 | `/index.html` | `200` |
| 404 | `/index.html` | `200` |

This allows React Router to work correctly when users directly load URLs such as:

```text
/products
/profile
/orders
```

Do not apply the fallback mechanism to:

- `/api/*`
- `/socket.io/*`

<!-- INSERT IMAGE HERE:
Capture the CloudFront Distribution after configuration.
The screenshot should show:
- Distribution Status: Deployed
- S3 Origin
- ALB Origin
- Behaviors:
  - /*
  - /api/*
  - /socket.io/*

Mask the Distribution ID, Account ID, and ALB DNS Name.
-->

![CloudFront Distribution](/images/5-Workshop/5.5-Frontend-deployment/create-cloudfront.png)

## Verification

- The CloudFront Distribution is **Deployed**.
- The S3 Origin uses Origin Access Control (OAC).
- The Bucket Policy allows only CloudFront to read objects.
- The Default Behavior (`/*`) points to the S3 Frontend Bucket.
- Behavior `/api/*` points to the Application Load Balancer and has caching disabled.
- Behavior `/socket.io/*` points to the Application Load Balancer and supports WebSocket traffic.
- Users cannot access the S3 bucket directly.
- React Router works correctly after a page reload.
- The REST API and Socket.IO work through CloudFront.

## Common issues

| Issue | Resolution |
|---|---|
| The API returns frontend `index.html` | The behavior path or order is incorrect; place dynamic behaviors before the default behavior |
| Socket.IO returns 400 or 502 | Check query and header forwarding, the origin protocol, and the ALB target |
| JWT is missing | Forward `Authorization` through the origin request policy |
| CloudFront returns S3 403 | Correct the OAC bucket policy and origin access settings |

## Summary

CloudFront is now the shared entry point for the frontend, API, and Socket.IO.


- Previous section: [5.5.2. Create the S3 Frontend Bucket](../5.5.2-Create-s3-frontend/)
- Next section: [5.5.4. Configure WAF](../5.5.4-Configure-waf/)
