---
title: "Blog 1"
date: 2024-01-01
weight: 1
chapter: false
pre: " <b> 3.1. </b> "
---

# [AWS Security] Solving the Edge Case: Combating SMS OTP Fraud with Vonage Network API and Amazon Cognito

Today I want to share notes on reducing SMS OTP fraud with **Vonage Network API** and **Amazon Cognito**.

SMS OTP has long been the backbone of many 2FA flows because it is convenient. Behind that convenience sit two operational pains: **SMS Toll Fraud / AIT** and **SIM Swap** attacks. Attackers can spam OTP traffic to inflate billing, or take over a phone number to bypass authentication.

Here are the practical points from an AWS + Vonage architecture that tries to address the problem at the root.

## 1. Why IP blocking / rate limiting alone is not enough

When OTP spam starts, the first reflex is often Rate Limit or IP blocking on **AWS WAF**. Modern botnets use many “clean” IPs and mimic real-user behavior.

What is missing is not only web-edge control, but carrier-side signal: **is this phone number safe before we send OTP?**

## 2. Foundation: Network APIs

The solution integrates CAMARA-standard Network APIs (Vonage / Ericsson) into the AWS auth flow. Two APIs stand out:

- **SIM Swap API:** silently checks with the mobile operator whether the SIM was recently swapped (often within 24–72 hours).
- **Number Verification API:** verifies whether the SIM currently using mobile data matches the claimed phone number — silent authentication with less friction than manual OTP entry.

## 3. Architecture: Amazon Cognito + Vonage

Vonage checks are injected into the **Amazon Cognito** flow through **Lambda Triggers**.

![Risk-Adaptive Customer Sign-In architecture](/images/3-BlogsPosted/blog1.png)

> Source: [AWS Study Group — Facebook](https://www.facebook.com/groups/awsstudygroupfcj/permalink/2198070834291210/)

### Flow (Layers 1 → 6)

1. **User & Edge:** Requests from Web/iOS/Android pass **CloudFront + AWS WAF** (rate limit, bot blocking, DDoS).
2. **Front door:** Clean traffic reaches **API Gateway + Auth Service (BFF)** to orchestrate sign-in.
3. **Identity + Risk:** **Amazon Cognito + Risk Engine** scores risk as LOW / MEDIUM / HIGH.
4. **Verification:**
   - Prefer **Passkey / FIDO2** when the device supports it.
   - Or call Vonage (**Identity Insights**, **Verify API**, **Fraud Defender**) plus carrier signals (SIM-swap, Silent Auth, SMS/Voice).

## 4. Takeaways

- **Proactive defense:** stop abuse early in the auth path instead of waiting for SMS billing spikes.
- **Better UX:** Number Verification can reduce drop-off during sign-up/sign-in.
- **Telecom-grade zero trust:** do not fully trust the client or displayed phone number; validate with the carrier too.

This is a useful reference when a CIAM system needs both stronger security and smoother authentication UX.
