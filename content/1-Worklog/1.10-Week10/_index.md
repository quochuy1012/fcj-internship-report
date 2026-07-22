---
title: "Week 10 Worklog"
date: 2024-01-01
weight: 10
chapter: false
pre: " <b> 1.10. </b> "
---

**Duration:** 17/07/2026 – 24/07/2026

### Week 10 Objectives

* Optimize architecture based on lecturer and mentor feedback.
* Add a Multi-AZ model for the VPC.
* Complete ECS outbound access (NAT Gateway / Elastic IP) for MongoDB Atlas; research VPC Endpoints as a future improvement.

### Tasks completed

| No. | Task | Start | End | Reference |
| --- | --- | --- | --- | --- |
| 1 | Optimize architecture from mentor/lecturer feedback | 17/07/2026 | 24/07/2026 | <https://hcm-rules.awsfcaj.com/3-project/> |
| 2 | Add Multi-AZ design for the VPC | 17/07/2026 | 24/07/2026 | <https://docs.aws.amazon.com/vpc/latest/userguide/vpc-subnets-availability-zones.html> |
| 3 | Configure NAT Gateway/EIP for ECS outbound and Atlas allowlisting; research VPC Endpoints | 17/07/2026 | 24/07/2026 | <https://docs.aws.amazon.com/vpc/latest/userguide/vpc-nat-gateway.html> |

### Achievements

* Revised the BravelSport architecture from mentor/lecturer feedback (high-level, public/private split).
* Extended the VPC across multiple AZs for higher availability.
* Private ECS reaches MongoDB Atlas through a NAT Gateway with a stable Elastic IP; VPC Endpoints noted as a later cost optimization.
