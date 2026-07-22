---
title: "Cấu hình Internet Gateway, NAT Gateway và Route Table"
date: 2026-07-19
weight: 3
chapter: false
pre: " <b> 5.3.3. </b> "
---

## Mục tiêu

Cho phép Public Subnet kết nối Internet và Private Subnet có outbound qua NAT Gateway mà không public ECS task.

## Các bước thực hiện

### 1. Internet Gateway

1. Tạo Internet Gateway tên `bravelsport-igw`.
2. Attach vào `bravelsport-vpc`.

### 2. Elastic IP và NAT Gateway

1. Allocate một Elastic IP trong Region chính.
2. Tạo NAT Gateway trong `nat-gateway-du-an`.
3. Gắn Elastic IP vừa cấp.
4. Chờ trạng thái `Available`.
5. Ghi lại Elastic IP để thêm vào MongoDB Atlas IP Access List; không công khai IP trong báo cáo nếu nhóm không muốn.

### 3. Public Route Table

1. Tạo `Ban_do_ao-rtb-public`.
2. Thêm route `0.0.0.0/0 → Internet Gateway`.
3. Associate với hai Public Subnet.

### 4. Private Route Table

1. Tạo `bravelsport-private-rt`.
2. Thêm route `0.0.0.0/0 → NAT Gateway`.
3. Associate với hai Private Subnet.


![Cấu hình routing - bước 1](/images/5-Workshop/5.3-Network-infrastructure/configure-routing1.png)

![Cấu hình routing - bước 2](/images/5-Workshop/5.3-Network-infrastructure/configure-routing2.png)

![Cấu hình routing - bước 3](/images/5-Workshop/5.3-Network-infrastructure/configure-routing3.png)

### Lưu ý

- Không vẽ hoặc mô tả NAT Gateway gửi traffic đến User.
- Luồng outbound đúng là `ECS → Private Route Table → NAT Gateway → IGW → external endpoint`.
- Một NAT Gateway là cấu hình tiết kiệm cho workshop, không phải high availability hoàn chỉnh.

## Kiểm tra kết quả

- IGW đã attach vào đúng VPC.
- NAT Gateway `Available` và nằm trong Public Subnet.
- Public RT có default route đến IGW.
- Private RT có default route đến NAT.
- Không gắn IGW trực tiếp cho ECS task.

## Lỗi thường gặp

| Lỗi | Nguyên nhân | Cách xử lý |
|---|---|---|
| NAT Pending lâu | EIP/subnet/IGW chưa đúng | Kiểm tra EIP và public route |
| ECS không pull image | Private route hoặc NAT lỗi | Kiểm tra route, SG egress và task stopped reason |
| Atlas từ chối kết nối | IP Access List chưa có NAT EIP | Thêm đúng EIP dạng `/32` |

## Tóm tắt

Routing đã cung cấp public ingress cho ALB và private outbound cho ECS.



- Phần trước: [5.3.2. Tạo subnet](../5.3.2-Create-subnets/)
- Phần tiếp theo: [5.3.4. Tạo Security Group](../5.3.4-Create-security-groups/)
