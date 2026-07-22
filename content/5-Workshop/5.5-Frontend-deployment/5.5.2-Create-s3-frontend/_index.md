---
title: "Create the S3 Frontend Bucket"
date: 2026-07-19
weight: 2
chapter: false
pre: " <b> 5.5.2. </b> "
---




## Objectives

Store the static frontend in a private Amazon S3 bucket and distribute it through Amazon CloudFront with Origin Access Control (OAC). Do not use S3 Static Website Hosting.

## Implementation steps

1. Create the bucket:

| Parameter | Value |
|---|---|
| Bucket name | `myapp-frontend-sport` |
| AWS Region | `ap-southeast-1 (Singapore)` |

2. Enable all **Block Public Access** settings.

3. Enable **Default Encryption (SSE-S3)** to encrypt data by default.

4. Enable **Bucket Versioning** if support for restoring previous website versions is required.

5. Do not enable **Static Website Hosting**, because the frontend will be accessed through CloudFront using Origin Access Control (OAC).

<!-- INSERT IMAGE HERE:
Capture the bucket Permissions tab.
The screenshot should show:
- Bucket name: myapp-frontend-sport
- Block Public Access: On

Mask the Account ID if displayed.
-->

![Create the S3 Frontend Bucket - step 1](/images/5-Workshop/5.5-Frontend-deployment/create-s3-frontend1.png)

6. Upload all contents of the `dist/` directory to the bucket:

```bash
aws s3 sync dist/ s3://myapp-frontend-sport/ --delete
```

This command will:

- Synchronize all contents of the `dist` directory.
- Automatically update modified files.
- Delete files that no longer exist in `dist`.

<!-- INSERT IMAGE HERE:
Capture the bucket Objects tab.
The screenshot should show:
- index.html
- assets/
- api/
- The uploaded frontend files

Mask the Account ID if displayed.
-->

![Create the S3 Frontend Bucket - step 2](/images/5-Workshop/5.5-Frontend-deployment/create-s3-frontend2.png)

7. Configure Cache-Control for objects in the bucket:

- `index.html`

```text
Cache-Control: no-cache, no-store, must-revalidate
```

- Hashed CSS, JavaScript, and image files in the `assets/` directory

```text
Cache-Control: public, max-age=31536000, immutable
```

This configuration ensures that:

- CloudFront always checks for a new version of `index.html`.
- Hashed static files are cached for a long period to improve performance and reduce request costs.

<!-- INSERT IMAGE HERE:
Capture Block Public Access and the object list. Mask the bucket ARN or owner and any unrelated object data.
Information to mask: Account ID, ARN, public IP address, token, cookie, secret, and user data if displayed.
-->



## Verification

- The bucket is private.
- `index.html` and the assets exist.
- Direct S3 object URLs are denied when the requester has no permission.
- No `.env` file or unplanned sensitive source map is present.

## Common issues

| Issue | Resolution |
|---|---|
| The website returns 403 through CloudFront | Configure OAC and the bucket policy in the next step |
| Files were uploaded at the wrong directory level | Synchronize the contents inside `dist/`, not a nested `dist` directory |
| The bucket is public | Re-enable Block Public Access and remove public ACLs or policies |

## Summary

The static frontend is stored securely in S3.


## Navigation

- Previous section: [5.5.1. Build the frontend](../5.5.1-Build-frontend/)
- Next section: [5.5.3. Create CloudFront](../5.5.3-Create-cloudfront/)
