---
title: "Tạo Target Group và Application Load Balancer"
date: 2026-07-19
weight: 4
chapter: false
pre: " <b> 5.4.4. </b> "
---


## Mục tiêu

Tạo Target Group và Application Load Balancer để tiếp nhận request từ CloudFront và chuyển tiếp đến ECS Fargate, đồng thời cấu hình health check cho backend.

## Các bước thực hiện

### 1. Tạo Target Group

Cấu hình Target Group như sau:

- Target type: `IP addresses`.
- Protocol: `HTTP`.
- Port: `3000`.
- VPC: `bravelsport-vpc`.
- Health check protocol: `HTTP`.
- Health check path: `/`.
- Success codes: `200`.
- Deregistration delay: `300 seconds`.
- Healthy threshold: `5`.
- Unhealthy threshold: `2`.
- Health check timeout: `5 seconds`.
- Health check interval: `30 seconds`.

Không đăng ký EC2 Build Machine vào Target Group vì backend được chạy trên Amazon ECS Fargate.

### 2. Tạo Application Load Balancer

Cấu hình Application Load Balancer:

- Scheme: `Internet-facing`.
- IP address type: `IPv4`.
- Chọn hai Public Subnet thuộc hai Availability Zone.
- Gắn Security Group `SG-ALB`.
- Listener: `HTTPS (443)`.
- Chọn ACM Certificate đã tạo trong Region `us-east-1`.
- Default action: Forward đến Target Group vừa tạo.

### 3. Hạn chế truy cập trực tiếp ALB

Để bảo đảm mọi request đều đi qua CloudFront và AWS WAF, áp dụng các biện pháp sau:

- Security Group của ALB chỉ cho phép lưu lượng từ CloudFront hoặc các nguồn được cho phép.
- CloudFront gửi Custom Origin Header đến ALB để xác thực request.
- Listener Rule của ALB chỉ chuyển tiếp request khi Custom Origin Header hợp lệ.

Không sử dụng ALB như điểm truy cập trực tiếp từ Internet nhằm tránh bỏ qua CloudFront và AWS WAF.

<!-- CHÈN ẢNH TẠI ĐÂY:
Chụp Application Load Balancer ở trạng thái Active.
Ảnh cần hiển thị Listener HTTPS 443 và Security Group.
Che DNS Name, ARN, Account ID và Certificate ARN trước khi đưa vào báo cáo.
-->

![Tạo Application Load Balancer - bước 1](/images/5-Workshop/5.4-Backend-deployment/create-alb1.png)

<!-- CHÈN ẢNH TẠI ĐÂY:
Chụp Target Group loại IP.
Ảnh cần hiển thị Port 3000, Health Check Path "/", Target Type IP.
Che ARN và Account ID nếu xuất hiện.
-->

![Tạo Target Group - bước 2](/images/5-Workshop/5.4-Backend-deployment/create-alb2.png)

## Kiểm tra kết quả

Sau khi hoàn thành, xác nhận:

- Application Load Balancer ở trạng thái **Active**.
- ALB được triển khai trên hai Public Subnet.
- Listener HTTPS `443` hoạt động bình thường.
- Target Group sử dụng **Target type = IP**.
- Backend sử dụng cổng `3000`.
- Health Check Path là `/`.
- Success Code là `200`.
- Không có EC2 Build Machine trong Target Group.
- ECS Task được đăng ký tự động vào Target Group sau khi Service khởi động.

## Lỗi thường gặp

| Lỗi | Cách xử lý |
|---|---|
| Chọn Target Type `Instance` | Tạo lại Target Group với Target Type `IP` |
| ALB không chuyển sang trạng thái Active | Kiểm tra Public Subnet, Internet Gateway, Route Table và Security Group |
| Target luôn Unhealthy | Kiểm tra backend có chạy cổng `3000` và endpoint `/` trả HTTP `200` |
| Health Check trả 404 | Kiểm tra lại Health Check Path hoặc tạo endpoint riêng |
| ALB có thể truy cập trực tiếp | Cấu hình Security Group hoặc Custom Origin Header để chỉ cho phép request từ CloudFront |

## Tóm tắt

Target Group và Application Load Balancer đã được cấu hình để tiếp nhận request từ CloudFront, kiểm tra trạng thái backend thông qua Health Check và chuyển tiếp request đến ECS Fargate Task trong Private Subnet.


- Phần trước: [5.4.3. Tạo IAM Role](../5.4.3-Create-iam-roles/)
- Phần tiếp theo: [5.4.5. Tạo ECS Cluster và Task Definition](../5.4.5-Create-ecs-cluster/)