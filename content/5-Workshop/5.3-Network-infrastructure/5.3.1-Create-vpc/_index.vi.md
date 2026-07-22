---
title: "Tạo VPC"
date: 2026-07-19
weight: 1
chapter: false
pre: " <b> 5.3.1. </b> "
---



## Mục tiêu

Tạo ranh giới mạng riêng cho ALB, NAT Gateway và ECS Fargate.

## Các bước thực hiện

1. Mở **VPC Console** tại Region 
2. Chọn **Create VPC** và chỉ tạo VPC trước.
3. Name tag đề xuất: `Ban_do_ao-vpc`.
4. IPv4 CIDR: `10.0.0.0/16`.
5. Không bật IPv6 nếu workshop chưa thiết kế IPv6.
6. Sau khi tạo, mở **Edit VPC settings**.
7. Bật **DNS resolution** và **DNS hostnames**.


![Tạo VPC BravelSport](/images/5-Workshop/5.3-Network-infrastructure/create-vpc.png)

## Kiểm tra kết quả

- Trạng thái VPC là `Available`.
- DNS resolution và DNS hostnames đều bật.
- CIDR không chồng lấn với mạng cần kết nối `10.0.0.0/16`.

## Lỗi thường gặp

| Lỗi | Nguyên nhân | Cách xử lý |
|---|---|---|
| CIDR không đủ subnet | Chọn CIDR quá nhỏ | Thiết kế lại trước khi tạo tài nguyên phụ thuộc |
| DNS hostname tắt | Bỏ qua VPC settings | Bật DNS resolution và DNS hostnames |
| Tạo nhầm Region | Console đang ở Region khác | Xóa tài nguyên thử nếu an toàn và tạo lại đúng Region |

## Tóm tắt

VPC BravelSport đã sẵn sàng để tạo subnet và routing.



- Phần trước: [5.3. Xây dựng hạ tầng mạng](../)
- Phần tiếp theo: [5.3.2. Tạo Public và Private Subnet](../5.3.2-Create-subnets/)
