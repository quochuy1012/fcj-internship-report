---
title: "Blog 3"
date: 2024-01-01
weight: 3
chapter: false
pre: " <b> 3.3. </b> "
---

# Modernizing SignalR with AWS AppSync Event API

I studied **Modernizing SignalR with AWS AppSync Event API** — how to modernize realtime for a .NET/React app by moving from traditional SignalR to the **AWS AppSync Event API**.

For realtime apps (order alerts, live dashboards, delivery status, chat, pub/sub), SignalR usually keeps WebSocket connections between server and clients. At scale, teams often struggle with connection management, always-on servers, sticky sessions, or a Redis backplane.

## New architecture

![AWS AppSync Event API architecture](/fcj-internship-report/images/3-BlogsPosted/blog3.png)

> Source: [AWS Study Group — Facebook](https://www.facebook.com/groups/awsstudygroupfcj/permalink/2206455833452710/)

Four main flows:

1. The **React Client** calls the **.NET API Server** when a realtime action happens.
2. The **.NET API Server** loads API keys/endpoints from **AWS Secrets Manager** instead of hard-coding them.
3. The backend publishes events to the **AWS AppSync Event API** over HTTP.
4. **AppSync** pushes realtime events to subscribers over WebSocket (pub/sub).

## Difference from traditional SignalR

- **SignalR:** the server handles both business logic and WebSocket connections.
- **AppSync Event API:** the server focuses on business logic and publishing events; AWS handles connections, fan-out, and scale.

This fits realtime notifications, live dashboards, order updates, collaborative apps, and IoT fan-out.

Secrets Manager keeps credentials out of source code. In practice you can also add IAM, CloudWatch, CloudTrail, and separate Dev/Staging/Prod environments.

## Conclusion

Modernizing SignalR does not mean dropping realtime. It means moving WebSocket infrastructure to a managed/serverless AWS service. The backend stays lean, the frontend still gets instant updates, and AWS owns the connection layer.
