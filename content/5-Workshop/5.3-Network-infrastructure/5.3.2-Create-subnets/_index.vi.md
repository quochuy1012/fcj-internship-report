---
title: "Tạo Public và Private Subnet"
date: 2026-07-19
weight: 2
chapter: false
pre: " <b> 5.3.2. </b> "
---




## Mục tiêu

Phân bố ALB/NAT Gateway và ECS Fargate trên hai Availability Zone.

## Các bước thực hiện

1. Trong VPC Console, chọn **Subnets → Create subnet**.
2. Chọn `bravelsport-vpc`.
3. Tạo bốn subnet với giá trị chưa xác nhận giữ dưới dạng placeholder:

| Subnet | Availability Zone | CIDR | Thành phần |
|---|---|---|---|
| `bravelsport-public-a` | `ap-southeast-1a` | `10.0.0.0/20` | ALB, NAT Gateway |
| `bravelsport-public-b` | `ap-southeast-1b` | `10.0.16.0/20` | ALB |
| `bravelsport-private-a` | `ap-southeast-1a` | `10.0.128.0/20` | ECS Fargate |
| `bravelsport-private-b` | `ap-southeast-1b` | `10.0.144.0/20` | ECS Fargate |


4. Public Subnet có thể bật auto-assign public IPv4 cho tài nguyên cần thiết; ECS không được đặt ở đây.
5. Private Subnet tắt auto-assign public IPv4.
6. Gắn tag phân biệt `Tier=Public` và `Tier=Private`.



![Danh sách subnet](/images/5-Workshop/5.3-Network-infrastructure/create-subnets.png)

## Kiểm tra kết quả

- Có đúng hai Public Subnet và hai Private Subnet.
- Hai subnet cùng loại nằm ở hai AZ khác nhau.
- Private Subnet không auto-assign public IPv4.

## Lỗi thường gặp

| Lỗi | Cách xử lý |
|---|---|
| Hai subnet dùng cùng AZ | Chọn lại AZ cho subnet B |
| CIDR overlap | Tính lại dải CIDR và tạo subnet mới |
| Private subnet bật public IP | Tắt auto-assign public IPv4 |

## Tóm tắt

Các subnet đã sẵn sàng cho routing, ALB và ECS Fargate.



- Phần trước: [5.3.1. Tạo VPC](../5.3.1-Create-vpc/)
- Phần tiếp theo: [5.3.3. Cấu hình routing](../5.3.3-Configure-routing/)
