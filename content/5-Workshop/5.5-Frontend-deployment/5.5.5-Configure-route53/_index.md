---
title: "Configure Route 53"
date: 2026-07-19
weight: 5
chapter: false
pre: " <b> 5.5.5. </b> "
---



## Objectives

Point the workshop domain to CloudFront and serve HTTPS using an ACM certificate in `us-east-1`.

## Implementation steps

1. In Amazon CloudFront, add the following **Alternate Domain Name (CNAME)**:

```text
bravelsport.com
```

2. Select an ACM Certificate with the following status:

```text
Issued
```

The certificate must be created in this Region:

```text
us-east-1 (N. Virginia)
```

because CloudFront supports ACM certificates only from this Region.

3. Choose **Save changes** and wait for the CloudFront Distribution to become:

```text
Deployed
```

4. Go to:

```text
Amazon Route 53 → Hosted Zones → bravelsport.com
```

5. Create the following record:

| Record | Type | Alias | Value |
|---|---|---|---|
| `bravelsport.com` | A | Yes | CloudFront Distribution |

If the website supports IPv6, also create:

| Record | Type | Alias | Value |
|---|---|---|---|
| `bravelsport.com` | AAAA | Yes | CloudFront Distribution |

6. Do not point user-facing DNS directly to:

- Amazon S3 Bucket
- Application Load Balancer
- Amazon ECS

All requests must pass through CloudFront to use:

- Origin Access Control (OAC)
- AWS WAF
- CloudFront Cache
- HTTPS

7. After the DNS update, test:

- `https://bravelsport.com`
- The frontend is loaded through CloudFront.
- The REST API works through `/api/*`.
- Socket.IO works through `/socket.io/*`.
- HTTPS works with the ACM Certificate.

<!-- INSERT IMAGE HERE:
Capture the Route 53 Hosted Zone.

The screenshot should show:

- Hosted Zone: bravelsport.com
- A Record (Alias) pointing to CloudFront
- NS Record
- SOA Record
- ACM validation CNAME, if present

Mask the Distribution Domain Name, Account ID, and unnecessary values.
-->


-->

![Configure the Route 53 alias](/images/5-Workshop/5.5-Frontend-deployment/configure-route53.png)

## Verification

```bash
nslookup <WORKSHOP_DOMAIN>
curl -I https://<WORKSHOP_DOMAIN>/
```

- DNS resolves to CloudFront.
- The HTTPS certificate is valid.
- The ALB DNS name is not exposed in the user-facing URL.

## Common issues

| Issue | Resolution |
|---|---|
| The certificate cannot be selected | Create the certificate in `us-east-1` and validate the domain |
| DNS still points to the old system | Check the record, TTL, and cutover plan |
| CloudFront returns 502 | Check the ALB origin, listener, Security Group, and target health |

## Summary

The domain now points to CloudFront, completing the entry path for the frontend and backend.

## Image inventory

| No. | File name | Image content | Placement |
|---:|---|---|---|
| 1 | `configure-route53.png` | Route 53 Alias to CloudFront | After creating the record |

## Navigation

- Previous section: [5.5.4. Configure WAF](../5.5.4-Configure-waf/)
- Next section: [5.6. Database and media](../../5.6-Data-and-media/)
