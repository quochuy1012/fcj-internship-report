---
title: "Build the React/Vite frontend"
date: 2026-07-19
weight: 1
chapter: false
pre: " <b> 5.5.1. </b> "
---



## Objectives

Configure the production URL and build static assets on the EC2 Build Machine.

## Implementation steps

1. On the build machine, either EC2 or a development machine:

```bash
cd ~/ShopBanDoAo/frontend
npm ci
```

2. Configure the API Base URL so that the frontend calls the backend through the same CloudFront domain:

```text
VITE_API_BASE_URL=https://bravelsport.com/api
```

When deploying with another domain name, replace `bravelsport.com` with the appropriate domain.

3. Configure the Socket.IO Client

The Socket.IO Client connects to the same website origin and uses the following namespace:

```text
/chat
```

Transport path:

```text
/socket.io
```

Do not configure the path as `/chat`, because `/chat` is only the Socket.IO namespace.

4. Build the frontend:

```bash
npm run build
```

After a successful build, inspect the output directory:

```bash
ls -la dist
```

The process creates a `dist/` directory containing HTML, CSS, and JavaScript files optimized for production.


5. Check that the bundle does not contain sensitive information:

```bash
rg -n "MONGO_URI|JWT_SECRET|API_SECRET|PRIVATE_KEY" dist || true
```

When the command returns no matches, the frontend bundle does not contain these secrets and is ready to be deployed to Amazon S3.

<!-- INSERT IMAGE HERE:
Capture the successful npm run build output and the dist directory. Mask any staging domain or token if displayed.
Information to mask: Account ID, ARN, public IP address, token, cookie, secret, and user data if displayed.
-->

![Build the React Vite frontend](/images/5-Workshop/5.5-Frontend-deployment/build-frontend.jpg)

## Verification

- `dist/` exists.
- The API does not point to `localhost:3000`.
- The Socket.IO path and namespace are configured correctly.
- The bundle contains no secrets.

## Common issues

| Issue | Resolution |
|---|---|
| The build uses old environment values | Delete `dist` and rebuild after updating the environment variables |
| The URL becomes `/api/api` | Check the base URL and endpoint paths in the Axios service |
| Socket.IO uses `/chat` as the path | Use `/chat` as the namespace and `/socket.io` as the transport path |

## Summary

The production frontend bundle is ready to be uploaded to S3.


- Previous section: [5.5. Frontend deployment](../)
- Next section: [5.5.2. Create the S3 Frontend Bucket](../5.5.2-Create-s3-frontend/)
