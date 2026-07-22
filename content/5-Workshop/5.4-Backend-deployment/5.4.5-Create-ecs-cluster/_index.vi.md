---
title: "Tạo CloudWatch Log Group, ECS Cluster và Task Definition"
date: 2026-07-19
weight: 5
chapter: false
pre: " <b> 5.4.5. </b> "
---



## Mục tiêu

Mô tả runtime container, IAM roles, port, environment, secret và logging trước khi tạo ECS Service.

## Các bước thực hiện

### 1. Tạo CloudWatch Log Group

Amazon ECS sử dụng CloudWatch Logs để lưu toàn bộ log của container backend. Việc tạo Log Group trước giúp Task Definition có thể ghi log ngay khi container khởi động.

Trong AWS Management Console, truy cập:

```text
CloudWatch → Logs → Log groups
```

Chọn:

```text
Create log group
```

Nhập các thông số sau:

| Thông số | Giá trị |
|---|---|
| Log group name | `/ecs/bravelsport-backend` |
| Log class | `Standard` |
| Retention | `30 days` |

Sau khi nhập đầy đủ thông tin, chọn **Create** để tạo Log Group.

Retention 30 ngày phù hợp với môi trường workshop vì vẫn giữ đủ log để kiểm tra lỗi nhưng không phát sinh chi phí lưu trữ quá lớn.

<!-- CHÈN ẢNH TẠI ĐÂY:
Chụp CloudWatch Log Group sau khi tạo.
Ảnh cần hiển thị:
- Log Group Name: /ecs/bravelsport-backend
- Retention: 30 days
- Log Class: Standard

Che Account ID và ARN nếu xuất hiện.
-->

![CloudWatch Log Group](/images/5-Workshop/5.4-Backend-deployment/cloudwatch-log-group.png)

CloudWatch Log Group này sẽ được sử dụng ở bước cấu hình Task Definition thông qua log driver `awslogs`, giúp ghi nhận toàn bộ log của ứng dụng NestJS trong quá trình vận hành.

### 2. Tạo ECS Cluster

- Cluster name: `bravelsport-cluster`.
- Infrastructure: AWS Fargate.
- Container Insights: Enabled.

### 3. Tạo Task Definition

- Launch type: AWS Fargate.
- Network mode: `awsvpc`.
- Task CPU: `512 CPU units (0.5 vCPU)`.
- Task memory: `1024 MiB (1 GiB)`.
- Task Execution Role: `ecsTaskExecutionRole`.
- Task Role: IAM Role của ứng dụng backend truy cập S3 Media.
- Operating system: Linux.
- CPU architecture: `X86_64`.

Container configuration:

- Container name: `backend-container`.
- Image URI: `<ECR_REPOSITORY_URI>:latest`.
- Essential: Yes.
- Container port: `3000/TCP`.
- Log driver: `awslogs`.
- Log group: `/ecs/bravelsport-backend`.
- Stream prefix: `ecs`.

<!-- CHÈN ẢNH TẠI ĐÂY:
Chụp image field đã che Account ID, port mapping 3000 và awslogs. Không hiển thị plain-text secret.
Thông tin cần che: Account ID, ARN, địa chỉ IP công khai, token, cookie, secret và dữ liệu người dùng nếu xuất hiện.
-->

![Container configuration](/images/5-Workshop/5.4-Backend-deployment/container-configuration.jpg)

Environment variables không nhạy cảm có thể khai báo trực tiếp trong Task Definition.

Các secret cần quản lý gồm `MONGO_URI`, `JWT_SECRET`, Google, VNPay, Cloudinary và email credentials nếu được sử dụng.

<!-- CHÈN ẢNH TẠI ĐÂY:
Chụp Task Definition family/revision, execution role, task role, CPU/memory và container. Che ARN, Account ID và secret references nếu cần.
Thông tin cần che: Account ID, ARN, địa chỉ IP công khai, token, cookie, secret và dữ liệu người dùng nếu xuất hiện.
-->

![ECS Task Definition](/images/5-Workshop/5.4-Backend-deployment/create-ecs-task-definition.png)

## Kiểm tra kết quả

- Task Definition là Fargate + `awsvpc`.
- Port mapping là `3000`.
- Execution Role và Task Role đúng vị trí.
- `awslogs` trỏ đúng Log Group.
- Không có secret plain text.

## Lỗi thường gặp

| Lỗi | Cách xử lý |
|---|---|
| CPU/memory không hợp lệ | Chọn cặp Fargate hỗ trợ; giữ placeholder khi chưa xác nhận |
| Image architecture mismatch | Rebuild image đúng platform hoặc đổi Task architecture |
| Log group not found | Tạo Log Group trước hoặc bật tùy chọn tạo tự động có quyền phù hợp |
| Secret ARN sai Region | Tạo secret/parameter cùng Region hoặc sửa reference |

## Tóm tắt

Cluster và Task Definition đã mô tả đầy đủ runtime backend.



- Phần trước: [5.4.4. Tạo ALB](../5.4.4-Create-alb/)
- Phần tiếp theo: [5.4.6. Tạo ECS Service](../5.4.6-Create-ecs-service/)
