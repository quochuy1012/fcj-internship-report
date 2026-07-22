---
title: "Create Public and Private Subnets"
date: 2026-07-19
weight: 2
chapter: false
pre: " <b> 5.3.2. </b> "
---




## Objectives

Distribute the ALB and NAT Gateway, as well as ECS Fargate, across two Availability Zones.

## Implementation steps

1. In the VPC Console, choose **Subnets → Create subnet**.
2. Select `bravelsport-vpc`.
3. Create four subnets using the following values:

| Subnet | Availability Zone | CIDR | Components |
|---|---|---|---|
| `bravelsport-public-a` | `ap-southeast-1a` | `10.0.0.0/20` | ALB, NAT Gateway |
| `bravelsport-public-b` | `ap-southeast-1b` | `10.0.16.0/20` | ALB |
| `bravelsport-private-a` | `ap-southeast-1a` | `10.0.128.0/20` | ECS Fargate |
| `bravelsport-private-b` | `ap-southeast-1b` | `10.0.144.0/20` | ECS Fargate |


4. Public Subnets may enable automatic public IPv4 assignment for resources that require it; ECS must not be placed in these subnets.
5. Disable automatic public IPv4 assignment for Private Subnets.
6. Add tags to distinguish them, such as `Tier=Public` and `Tier=Private`.



![Subnet list](/images/5-Workshop/5.3-Network-infrastructure/create-subnets.png)

## Verification

- Exactly two Public Subnets and two Private Subnets exist.
- The two subnets of each type are in different AZs.
- Private Subnets do not automatically assign public IPv4 addresses.

## Common issues

| Issue | Resolution |
|---|---|
| Both subnets use the same AZ | Select another AZ for subnet B |
| CIDR ranges overlap | Recalculate the CIDR ranges and create new subnets |
| A Private Subnet assigns public IP addresses | Disable automatic public IPv4 assignment |

## Summary

The subnets are ready for routing, the ALB, and ECS Fargate.



- Previous section: [5.3.1. Create a VPC](../5.3.1-Create-vpc/)
- Next section: [5.3.3. Configure routing](../5.3.3-Configure-routing/)
