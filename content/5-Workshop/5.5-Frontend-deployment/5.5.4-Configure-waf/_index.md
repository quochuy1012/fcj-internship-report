---
title: "Configure AWS WAF"
date: 2026-07-19
weight: 4
chapter: false
pre: " <b> 5.5.4. </b> "
---



## Objectives

Associate an AWS WAF Web ACL with Amazon CloudFront to protect the frontend and backend from malicious requests before they reach the Application Load Balancer or Amazon S3.

## Implementation steps

1. Open **AWS WAF & Shield**.

2. Select the scope:

```text
CloudFront (Global)
```

3. Create the Web ACL:

```text
CreatedByCloudFront-7306abc9
```

After creation, associate the Web ACL with the CloudFront Distribution.

4. Add AWS Managed Rules.

This workshop uses the following common Managed Rule groups:

- AWSManagedRulesCommonRuleSet
- AWSManagedRulesKnownBadInputsRuleSet
- AWSManagedRulesAmazonIpReputationList

5. During testing, rules that may produce false positives can be configured in:

```text
Count
```

After confirming that the application operates correctly, change them to:

```text
Block
```

6. Enable:

- Sampled Requests
- CloudWatch Metrics

to monitor traffic and inspect requests processed by WAF.

7. Test the following system functions:

- Sign in.
- Registration.
- REST API.
- Image upload.
- Socket.IO.
- React Router.

Ensure that valid requests are not blocked by WAF before changing all rules to **Block** mode.

Do not apply production blocking rules until testing has been completed.

<!-- INSERT IMAGE HERE:
Capture the Web ACL after creation.
The screenshot should show:
- Web ACL Name: CreatedByCloudFront-7306abc9
- Scope: CloudFront
- Associated CloudFront Distribution
- 3 AWS Managed Rules

Mask the Account ID, ARN, and Distribution ID.
-->

![AWS WAF Web ACL](/images/5-Workshop/5.5-Frontend-deployment/configure-waf.png)

## Verification

- The Web ACL is associated with the CloudFront Distribution.
- The scope is **CloudFront (Global)**.
- The Web ACL contains **3 AWS Managed Rules**.
- Valid requests can still access the website normally.
- Sampled Requests is working.
- CloudWatch Metrics are recorded.
- Rules are changed to **Block** only after testing is complete.
## Common issues

| Issue | Resolution |
|---|---|
| Login or upload requests are blocked | Review sampled requests and adjust rules or exclusions |
| The distribution is not listed | Check that the scope is CloudFront Global |
| Costs increase | Limit the number of rules and monitor request volume |

## Summary

WAF has been deployed using an observe-first, block-later approach.



- Previous section: [5.5.3. Create CloudFront](../5.5.3-Create-cloudfront/)
- Next section: [5.5.5. Configure Route 53](../5.5.5-Configure-route53/)
