---
title: "Workshop"
date: 2026-07-19
weight: 5
chapter: false
pre: " <b> 5. </b> "
---

# Deploying BravelSport on AWS

BravelSport is an e-commerce and sports facility booking platform. The source includes a React/Vite frontend and a NestJS backend using MongoDB (Mongoose), JWT auth, and Socket.IO for realtime chat.

This workshop describes the **target architecture and hands-on deployment process**. It does not claim the live site already runs the full AWS architecture unless Console evidence is available.

## Target architecture

- Frontend is built as static files in a private S3 Frontend Bucket and served by CloudFront.
- AWS WAF is associated with CloudFront at the edge.
- CloudFront forwards `/api/*` and `/socket.io/*` to an Application Load Balancer.
- The ALB forwards traffic through an `ip` Target Group to ECS Fargate on port `3000`.
- Socket.IO uses `/socket.io/*`; chat uses the `/chat` namespace.
- ECS Fargate runs in Private Subnets without public IPs.
- ECS reaches MongoDB Atlas via NAT Gateway and Internet Gateway.
- An EC2 Ubuntu instance is only a build/deploy machine — not production ingress.
- S3 Media is a workshop extension (current code also supports Cloudinary / local `/uploads`).

![BravelSport AWS Architecture](/images/5-Workshop/architecture/bravelsport-aws-architecture.png)

## Workshop contents

1. [5.1. Workshop overview](./5.1-Workshop-overview/)
2. [5.2. Prerequisites](./5.2-Prerequisite/)
3. [5.3. Building the network infrastructure](./5.3-Network-infrastructure/)
4. [5.4. Backend deployment](./5.4-Backend-deployment/)
5. [5.5. Frontend deployment](./5.5-Frontend-deployment/)
6. [5.6. Database and media](./5.6-Data-and-media/)
7. [5.7. End-to-end testing](./5.7-Testing/)
8. [5.8. Resource cleanup](./5.8-Cleanup/)

### Notes

Do not include access keys, secrets, MongoDB connection strings, JWT secrets, auth cookies, or full Account IDs. Unconfirmed values stay as `[TO BE CONFIRMED]`.

## Verification

- Hugo menu shows sections 5.1–5.8 in order.
- Every image path starts with `/images/5-Workshop/`.
- Pages clearly separate the EC2 Build Machine from the ECS Fargate runtime.

## Common issues

| Issue | Cause | Resolution |
|---|---|---|
| Wrong menu order | Bad `weight` / `pre` | Fix front matter |
| Images missing | Wrong `static/` path | Match `/images/5-Workshop/...` |
| Broken child links | Renamed folders | Check relative links |

## Summary

This workshop covers preparation, networking, backend/frontend deploy, database/media, testing, and cleanup.

## Navigation

- Next: [5.1. Workshop overview](./5.1-Workshop-overview/)
