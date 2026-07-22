---
title: "Configure the Internet Gateway, NAT Gateway, and Route Tables"
date: 2026-07-19
weight: 3
chapter: false
pre: " <b> 5.3.3. </b> "
---

## Objectives

Allow Public Subnets to connect to the Internet and provide outbound access for Private Subnets through a NAT Gateway without exposing ECS tasks publicly.

## Implementation steps

### 1. Internet Gateway

1. Create an Internet Gateway named `bravelsport-igw`.
2. Attach it to `bravelsport-vpc`.

### 2. Elastic IP and NAT Gateway

1. Allocate an Elastic IP in the primary Region.
2. Create a NAT Gateway in `nat-gateway-du-an`.
3. Associate the newly allocated Elastic IP.
4. Wait until the status becomes `Available`.
5. Record the Elastic IP so that it can be added to the MongoDB Atlas IP Access List; do not publish the IP in the report if the team prefers to keep it private.

### 3. Public Route Table

1. Create `Ban_do_ao-rtb-public`.
2. Add the route `0.0.0.0/0 → Internet Gateway`.
3. Associate it with both Public Subnets.

### 4. Private Route Table

1. Create `bravelsport-private-rt`.
2. Add the route `0.0.0.0/0 → NAT Gateway`.
3. Associate it with both Private Subnets.


![Routing configuration - step 1](/images/5-Workshop/5.3-Network-infrastructure/configure-routing1.png)

![Routing configuration - step 2](/images/5-Workshop/5.3-Network-infrastructure/configure-routing2.png)

![Routing configuration - step 3](/images/5-Workshop/5.3-Network-infrastructure/configure-routing3.png)

### Notes

- Do not draw or describe the NAT Gateway as sending traffic to the User.
- The correct outbound flow is `ECS → Private Route Table → NAT Gateway → IGW → external endpoint`.
- A single NAT Gateway is a cost-saving workshop configuration, not a fully highly available design.

## Verification

- The IGW is attached to the correct VPC.
- The NAT Gateway is `Available` and located in a Public Subnet.
- The Public Route Table has a default route to the IGW.
- The Private Route Table has a default route to the NAT Gateway.
- The IGW is not attached directly to ECS tasks.

## Common issues

| Issue | Cause | Resolution |
|---|---|---|
| The NAT Gateway remains Pending for a long time | The EIP, subnet, or IGW configuration is incorrect | Check the EIP and the public route |
| ECS cannot pull an image | The private route or NAT Gateway is misconfigured | Check the route, Security Group egress, and task stopped reason |
| Atlas rejects the connection | The NAT EIP is missing from the IP Access List | Add the correct EIP as a `/32` entry |

## Summary

The routing configuration provides public ingress for the ALB and private outbound access for ECS.



- Previous section: [5.3.2. Create subnets](../5.3.2-Create-subnets/)
- Next section: [5.3.4. Create Security Groups](../5.3.4-Create-security-groups/)
