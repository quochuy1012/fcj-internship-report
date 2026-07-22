---
title: "Building the network infrastructure"
date: 2026-07-19
weight: 3
chapter: false
pre: " <b> 5.3. </b> "
---


## Objectives

Create the network foundation so that the ALB operates in Public Subnets, ECS Fargate runs in Private Subnets, and tasks have outbound connectivity through a NAT Gateway without being assigned public IP addresses.

## Target network design

The workshop uses:

- One VPC.
- Two Public Subnets, each in a different Availability Zone.
- Two Private Subnets, each in a different Availability Zone.
- One Internet Gateway.
- One NAT Gateway associated with an Elastic IP in a Public Subnet.
- One Public Route Table and one Private Route Table.
- SG-ALB, SG-ECS, and an SG for the EC2 Build Machine.

```text
CloudFront
   |
   v
ALB -- Public Subnet A/B
   |
   v
ECS Fargate -- Private Subnet A/B
   |
   v
Private Route Table → NAT Gateway → Internet Gateway → MongoDB Atlas/public AWS endpoints
```



![VPC resource map](/images/5-Workshop/5.3-Network-infrastructure/vpc-resource-map.png)

## Implementation steps

1. [5.3.1. Create a VPC](./5.3.1-Create-vpc/)
2. [5.3.2. Create Public and Private Subnets](./5.3.2-Create-subnets/)
3. [5.3.3. Configure the Internet Gateway, NAT Gateway, and Route Tables](./5.3.3-Configure-routing/)
4. [5.3.4. Create Security Groups](./5.3.4-Create-security-groups/)

### Availability note

Using a single NAT Gateway reduces workshop costs but creates a single point of failure for outbound traffic from the Private Subnets. A fully highly available design normally deploys one NAT Gateway per Availability Zone and routes each private subnet to the NAT Gateway in the same AZ. This workshop does not claim that a single-NAT configuration is production-ready.

## Verification

- DNS resolution and DNS hostnames are enabled for the VPC.
- The ALB can be placed in two Public Subnets.
- The ECS Service can use two Private Subnets.
- The Public Route Table has an Internet route through the IGW.
- The Private Route Table has a default route through the NAT Gateway.
- ECS tasks do not have public IP addresses.

## Common issues

| Issue | Resolution |
|---|---|
| The NAT Gateway was created in a Private Subnet | Recreate the NAT Gateway in a Public Subnet that has a route to the IGW |
| A Private Subnet cannot access the Internet | Check the route to the NAT Gateway, confirm the NAT Gateway is `Available`, and verify its EIP |
| The ALB can use only one AZ | Create a second Public Subnet in another AZ |
| ECS receives traffic directly | Remove the `0.0.0.0/0:3000` rule and use only SG-ALB as the source |

## Summary

The network layer separates the public entry point from the private runtime and provides a fixed outbound IP address for MongoDB Atlas.



- Previous section: [5.2. Prerequisites](../5.2-Prerequisite/)
- Next section: [5.3.1. Create a VPC](./5.3.1-Create-vpc/)
