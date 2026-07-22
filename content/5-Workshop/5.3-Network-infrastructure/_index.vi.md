---
title: "Xây dựng hạ tầng mạng"
date: 2026-07-19
weight: 3
chapter: false
pre: " <b> 5.3. </b> "
---


## Mục tiêu

Tạo network foundation để ALB hoạt động trong Public Subnet, ECS Fargate chạy trong Private Subnet và có outbound qua NAT Gateway mà không gán public IP cho task.

## Thiết kế mạng mục tiêu

Workshop sử dụng:

- Một VPC.
- Hai Public Subnet, mỗi subnet thuộc một Availability Zone.
- Hai Private Subnet, mỗi subnet thuộc một Availability Zone.
- Một Internet Gateway.
- Một NAT Gateway gắn Elastic IP trong một Public Subnet.
- Một Public Route Table và một Private Route Table.
- SG-ALB, SG-ECS và SG-EC2 Build Machine.

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

## Các bước thực hiện

1. [5.3.1. Tạo VPC](./5.3.1-Create-vpc/)
2. [5.3.2. Tạo Public và Private Subnet](./5.3.2-Create-subnets/)
3. [5.3.3. Cấu hình Internet Gateway, NAT Gateway và Route Table](./5.3.3-Configure-routing/)
4. [5.3.4. Tạo Security Group](./5.3.4-Create-security-groups/)

### Lưu ý về tính sẵn sàng

Một NAT Gateway duy nhất phù hợp để giảm chi phí trong workshop nhưng tạo single point of failure cho outbound của các Private Subnet. Thiết kế high availability đầy đủ thường bố trí NAT Gateway theo từng Availability Zone và route private subnet đến NAT cùng AZ. Workshop không khẳng định cấu hình một NAT là production-ready.

## Kiểm tra kết quả

- VPC bật DNS resolution và DNS hostnames.
- ALB có thể được chọn ở hai Public Subnet.
- ECS Service có thể chọn hai Private Subnet.
- Public Route Table có route Internet qua IGW.
- Private Route Table có default route qua NAT Gateway.
- ECS không có public IP.

## Lỗi thường gặp

| Lỗi | Cách xử lý |
|---|---|
| NAT Gateway tạo trong Private Subnet | Tạo lại NAT trong Public Subnet có route đến IGW |
| Private Subnet không ra Internet | Kiểm tra route đến NAT, NAT trạng thái Available và EIP |
| ALB chỉ chọn được một AZ | Tạo Public Subnet thứ hai ở AZ khác |
| ECS nhận traffic trực tiếp | Xóa rule `0.0.0.0/0:3000`, chỉ dùng SG-ALB làm source |

## Tóm tắt

Network layer đã tách public entry và private runtime, đồng thời cung cấp outbound có IP cố định cho MongoDB Atlas.



- Phần trước: [5.2. Các bước chuẩn bị](../5.2-Prerequisite/)
- Phần tiếp theo: [5.3.1. Tạo VPC](./5.3.1-Create-vpc/)
