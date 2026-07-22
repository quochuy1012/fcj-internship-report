---
title: "Create a VPC"
date: 2026-07-19
weight: 1
chapter: false
pre: " <b> 5.3.1. </b> "
---



## Objectives

Create a private network boundary for the ALB, NAT Gateway, and ECS Fargate.

## Implementation steps

1. Open the **VPC Console** in the selected Region.
2. Choose **Create VPC** and initially create only the VPC.
3. Suggested name tag: `Ban_do_ao-vpc`.
4. IPv4 CIDR: `10.0.0.0/16`.
5. Do not enable IPv6 unless the workshop includes an IPv6 design.
6. After creation, open **Edit VPC settings**.
7. Enable **DNS resolution** and **DNS hostnames**.


![Create the BravelSport VPC](/images/5-Workshop/5.3-Network-infrastructure/create-vpc.png)

## Verification

- The VPC status is `Available`.
- DNS resolution and DNS hostnames are enabled.
- The `10.0.0.0/16` CIDR does not overlap with any network that must be connected.

## Common issues

| Issue | Cause | Resolution |
|---|---|---|
| The CIDR does not provide enough subnets | The selected CIDR is too small | Redesign it before creating dependent resources |
| DNS hostnames are disabled | VPC settings were skipped | Enable DNS resolution and DNS hostnames |
| The VPC was created in the wrong Region | The console was set to another Region | Delete the test resource if it is safe to do so and recreate it in the correct Region |

## Summary

The BravelSport VPC is ready for subnet and routing configuration.



- Previous section: [5.3. Building the network infrastructure](../)
- Next section: [5.3.2. Create Public and Private Subnets](../5.3.2-Create-subnets/)
