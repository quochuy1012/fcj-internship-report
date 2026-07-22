---
title: "Tạo ECS Service"
date: 2026-07-19
weight: 6
chapter: false
pre: " <b> 5.4.6. </b> "
---




## Mục tiêu

Chạy backend trong Private Subnet, không public IP và đăng ký task vào Target Group loại `ip`.

## Các bước thực hiện

### 1. Tạo ECS Service

Trong Amazon ECS Console, truy cập:

```text
Amazon ECS → Clusters → web-ban-ao-cluster
```

Chọn **Create service**.

Cấu hình Service như sau:

| Thông số | Giá trị |
|---|---|
| Cluster | `web-ban-ao-cluster` |
| Launch type | AWS Fargate |
| Task Definition | `web-ban-ao-backend-task` |
| Revision | Revision mới nhất |
| Service name | `web-ban-ao-backend-service` |
| Desired tasks | `1` |
| Deployment type | Rolling update |

### 2. Cấu hình Networking

Thiết lập mạng cho ECS Service:

- VPC: `bravelsport-vpc`
- Subnets: `bravelsport-private-a` và `bravelsport-private-b`
- Security Group: `SG-ECS`
- Auto-assign public IP: **Disabled**

ECS Task sẽ chạy hoàn toàn trong Private Subnet và truy cập Internet thông qua NAT Gateway.

### 3. Cấu hình Load Balancer

Chọn:

- Load balancer type: **Application Load Balancer**
- Application Load Balancer: ALB đã tạo
- Container: `backend-container`
- Container port: `3000`
- Target Group: `bravelsport-backend-tg`
- Target type: `IP`

Health Check Grace Period:

```text
60 seconds
```

### 4. Tạo Service

Sau khi hoàn tất cấu hình, chọn **Create**.

Amazon ECS sẽ:

- Khởi tạo Task từ Task Definition.
- Đăng ký Task vào Target Group.
- Thực hiện Health Check.
- Chuyển Service sang trạng thái **Active** khi backend hoạt động bình thường.

<!-- CHÈN ẢNH TẠI ĐÂY:
Chụp danh sách ECS Service.
Ảnh cần hiển thị:
- Cluster web-ban-ao-cluster
- Service web-ban-ao-backend-service
- Status Active
- Launch Type Fargate
- Desired Tasks = 1
- Deployment Completed

Che Account ID và ARN nếu xuất hiện.
-->

![ECS Service](/images/5-Workshop/5.4-Backend-deployment/create-ecs-service.png)

<!-- CHÈN ẢNH TẠI ĐÂY:
Chụp trang chi tiết Service.
Ảnh cần hiển thị:
- Service Overview
- Status Active
- Tasks (1 Desired)
- Deployment Success
- Task Definition Revision
- Running Task

Che Task ID, Account ID và ARN nếu cần.
-->

![ECS Service Overview](/images/5-Workshop/5.4-Backend-deployment/ecs-service-overview.png)

## Kiểm tra kết quả

Sau khi hoàn thành, xác nhận:

- ECS Service ở trạng thái **Active**.
- Deployment status là **Success**.
- Desired Tasks là **1**.
- Running Tasks là **1**.
- Launch Type là **AWS Fargate**.
- Task Definition sử dụng revision mới nhất.
- ECS Task đã được đăng ký vào Target Group.
- Health Check của ALB thành công.

<!-- CHÈN ẢNH TẠI ĐÂY:
Chụp service với VPC, private subnets, public IP off và Target Group. Che ARN, subnet ID và service discovery data nếu có.
Thông tin cần che: Account ID, ARN, địa chỉ IP công khai, token, cookie, secret và dữ liệu người dùng nếu xuất hiện.
-->

![Tạo ECS Service](/images/5-Workshop/5.4-Backend-deployment/create-ecs-service.png)

### Kiểm tra task

Mở tab Tasks và xác nhận task ở trạng thái `Running`.

<!-- CHÈN ẢNH TẠI ĐÂY:
Chụp task Running. Che task ARN, private IP và Account ID.
Thông tin cần che: Account ID, ARN, địa chỉ IP công khai, token, cookie, secret và dữ liệu người dùng nếu xuất hiện.
-->

![ECS Task Running](/images/5-Workshop/5.4-Backend-deployment/ecs-task-running.jpg)

### Kiểm tra Target Group

Chờ target chuyển sang `Healthy`.

<!-- CHÈN ẢNH TẠI ĐÂY:
Chụp target health status Healthy. Che target private IP và resource ID.
Thông tin cần che: Account ID, ARN, địa chỉ IP công khai, token, cookie, secret và dữ liệu người dùng nếu xuất hiện.
-->

![Healthy ECS target](/images/5-Workshop/5.4-Backend-deployment/ecs-target-healthy.png)

### Kiểm tra log

Mở CloudWatch Logs và xác nhận có log startup, port `3000` và kết nối database nếu đã cấu hình. Không chụp secret hoặc full connection string.

<!-- CHÈN ẢNH TẠI ĐÂY:
Chụp log stream của task. Che user data, JWT, email, payment payload và database URI.
Thông tin cần che: Account ID, ARN, địa chỉ IP công khai, token, cookie, secret và dữ liệu người dùng nếu xuất hiện.
-->

![Container logs](/images/5-Workshop/5.4-Backend-deployment/container-logs.jpg)

## Kiểm tra kết quả

- Service stable và running count bằng desired count.
- Task nằm trong Private Subnet, public IP off.
- Target Healthy.
- Log xuất hiện trong CloudWatch.
- ALB không nối đến EC2 Build Machine.

## Lỗi thường gặp

| Lỗi | Cách xử lý |
|---|---|
| Task dừng | Xem stopped reason, execution role, image URI và log |
| Target unhealthy | Kiểm tra health path `/`, SG và port `3000` |
| Không kết nối Atlas | Kiểm tra NAT EIP trong IP Access List và `MONGO_URI` |
| Service không tạo được | Kiểm tra `iam:PassRole`, subnet và target group |

## Tóm tắt

Backend BravelSport đã chạy trên ECS Fargate và được ALB kiểm tra sức khỏe.


- Phần trước: [5.4.5. Tạo ECS Cluster và Task Definition](../5.4.5-Create-ecs-cluster/)
- Phần tiếp theo: [5.5. Triển khai frontend](../../5.5-Frontend-deployment/)
