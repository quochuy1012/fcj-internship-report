---
title: "Worklog Tuần 10"
date: 2024-01-01
weight: 10
chapter: false
pre: " <b> 1.10. </b> "
---

**Thời gian:** 17/07/2026 – 24/07/2026

### Mục tiêu tuần 10

* Tối ưu kiến trúc theo góp ý của giảng viên và mentor.
* Bổ sung mô hình Multi-AZ cho VPC.
* Hoàn thiện outbound cho ECS (NAT Gateway / Elastic IP) để kết nối MongoDB Atlas; nghiên cứu VPC Endpoint như hướng cải tiến.

### Công việc đã làm

| STT | Công việc | Ngày bắt đầu | Ngày hoàn thành | Tài liệu tham khảo |
| --- | --- | --- | --- | --- |
| 1 | Tối ưu kiến trúc theo góp ý giảng viên và mentor | 17/07/2026 | 24/07/2026 | <https://hcm-rules.awsfcaj.com/3-project/> |
| 2 | Bổ sung mô hình Multi-AZ cho VPC | 17/07/2026 | 24/07/2026 | <https://docs.aws.amazon.com/vpc/latest/userguide/vpc-subnets-availability-zones.html> |
| 3 | Cấu hình NAT Gateway/EIP cho ECS outbound và Atlas allowlist; nghiên cứu VPC Endpoint | 17/07/2026 | 24/07/2026 | <https://docs.aws.amazon.com/vpc/latest/userguide/vpc-nat-gateway.html> |

### Kết quả đạt được

* Chỉnh kiến trúc BravelSport theo góp ý mentor/giảng viên (high-level, tách public/private).
* Mở rộng VPC Multi-AZ để tăng tính sẵn sàng.
* ECS private truy cập MongoDB Atlas qua NAT Gateway với Elastic IP cố định; ghi nhận VPC Endpoint là hướng tối ưu chi phí sau này.
