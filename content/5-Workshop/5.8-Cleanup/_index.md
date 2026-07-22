---
title: "Resource cleanup"
date: 2026-07-19
weight: 8
chapter: false
pre: " <b> 5.8. </b> "
---


# Resource cleanup

## Objectives

Delete workshop resources in an order that reduces dependencies and prevents further costs.

### Notes

Do not delete a Hosted Zone, domain, CloudFront distribution, bucket, database, or record serving a real website without a backup, migration plan, and approval. Before every action, confirm the Account, Region, Project, and Environment.

## Implementation steps

Recommended order:

1. Delete Route 53 records used only for the workshop.
2. Disable and delete the CloudFront Distribution.
3. Remove the WAF association and delete the Web ACL or rules used only for the workshop.
4. Scale the ECS Service to `0` first when an observation period is required.
5. Delete the ECS Service.
6. Confirm that ECS tasks have stopped.
7. Delete the ALB.
8. Delete the Target Group.
9. Delete the ECS Cluster when no service or task remains.
10. Deregister or delete Task Definition revisions when required.
11. Delete ECR images and the repository.
12. Delete the CloudWatch Log Group.
13. Empty and delete the S3 Frontend Bucket.
14. Empty and delete the S3 Media Bucket.
15. Delete the NAT Gateway and wait until its status becomes `Deleted`.
16. Release the Elastic IP after the NAT Gateway has been deleted.
17. Terminate the EC2 Build Machine and check associated EBS volumes and snapshots.
18. Delete custom route tables after removing their associations.
19. Delete Security Groups after no dependent ENIs remain.
20. Delete the subnets.
21. Detach and delete the Internet Gateway.
22. Delete the VPC.
23. Delete IAM Roles and Policies used only for the workshop after confirming that no resources reference them.
24. Review the Route 53 Hosted Zone, domain registration, and Atlas resources; delete them only when they are actually within the workshop scope.

<!-- INSERT IMAGE HERE:
Capture the NAT Gateway in the Deleting or Deleted state. Mask the NAT ID and Elastic IP.
Information to mask: Account ID, ARN, public IP address, token, cookie, secret, and user data if displayed.
-->

<!-- Image not yet available under static: /fcj-internship-report/images/5-Workshop/5.8-Cleanup/delete-nat-gateway.png -->
<!-- INSERT IMAGE HERE:
Capture the EC2 Build Machine in the Terminated state. Mask the Instance ID and IP address.
Information to mask: Account ID, ARN, public IP address, token, cookie, secret, and user data if displayed.
-->

<!-- Image not yet available under static: /fcj-internship-report/images/5-Workshop/5.8-Cleanup/ec2-terminated.png -->

## Verification

- No ECS service or task remains running.
- No ALB or NAT Gateway remains.
- The Elastic IP has been released.
- Workshop S3 buckets have been deleted or retained for a documented reason.
- EC2 has been terminated and no unplanned EBS volume remains.
- The VPC resource map has no remaining dependencies.
- Workshop IAM roles are no longer in use.
- After billing data has updated, Cost Explorer or Billing shows no unplanned running resources.

<!-- INSERT IMAGE HERE:
Capture the VPC resource map or Resource Explorer showing that workshop resources are gone; Cost Explorer can be added after billing data updates. Mask the Account ID and sensitive cost information.
Information to mask: Account ID, ARN, public IP address, token, cookie, secret, and user data if displayed.
-->

<!-- Image not yet available under static: /fcj-internship-report/images/5-Workshop/5.8-Cleanup/verify-cleanup.png -->

## Common issues

| Issue | Cause | Resolution |
|---|---|---|
| The VPC cannot be deleted | An ENI, NAT Gateway, ALB, or endpoint still exists | Check Network Interfaces and dependencies |
| A Security Group cannot be deleted | An ENI or Security Group reference still exists | Delete dependent resources first |
| A bucket cannot be deleted | Objects, versions, or delete markers remain | Empty all object versions |
| The EIP still incurs charges | It was not released after the NAT Gateway was deleted | Release the Elastic IP |
| CloudFront cannot be deleted | The Distribution has not been disabled and deployed | Disable it, wait until the change is deployed, and then delete it |

## Summary

Workshop resources have been deleted in a safe order, and residual costs have been reviewed.

## Image inventory

| No. | File name | Image content | Placement |
|---:|---|---|---|
| 1 | `delete-nat-gateway.png` | NAT Gateway after changing to Deleted | After the NAT deletion step |
| 2 | `verify-cleanup.png` | VPC or resource list and Billing after cleanup | At the end of the chapter |
| 3 | `ec2-terminated.png` | Terminated EC2 Build Machine | After the EC2 deletion step |

## Navigation

- Previous section: [5.7. End-to-end testing](../5.7-Testing/)
- Next section: [Workshop](../)
