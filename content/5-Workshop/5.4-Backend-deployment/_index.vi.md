---
title: "Triển khai backend NestJS trên ECS Fargate"
date: 2026-07-19
weight: 4
chapter: false
pre: " <b> 5.4. </b> "
---


## Mục tiêu

Build backend trên EC2, lưu image trong ECR, cấu hình IAM và CloudWatch, sau đó chạy container bằng ECS Fargate sau ALB/Target Group loại `ip`.

## Các bước thực hiện

1. [5.4.1. Tạo Amazon ECR Repository](./5.4.1-Create-ecr/)
2. [5.4.2. Build và push Docker image](./5.4.2-Build-and-push-image/)
3. [5.4.3. Tạo IAM Role](./5.4.3-Create-iam-roles/)
4. [5.4.4. Tạo Target Group và ALB](./5.4.4-Create-alb/)
5. [5.4.5. Tạo CloudWatch Log Group, ECS Cluster và Task Definition](./5.4.5-Create-ecs-cluster/)
6. [5.4.6. Tạo ECS Service](./5.4.6-Create-ecs-service/)


![Backend deployment overview](/images/5-Workshop/5.4-Backend-deployment/backend-deployment-overview.png)

## Nguyên tắc

- EC2 không chạy backend production.
- Target Group phải dùng target type `ip` cho Fargate `awsvpc`.
- Container port là `3000`.
- Health check path ban đầu là `/` và phải được xác nhận bằng response thật.
- Secret mechanism: `AWS Systems Manager Parameter Store (SecureString)`.
- Không lưu MongoDB URI hoặc JWT secret trong IAM, source code hay Docker image.

## Kiểm tra kết quả

- ECR có image digest và tag đã chọn.
- Task Definition dùng đúng image URI, execution role, task role và log configuration.
- ECS task nằm trong Private Subnet, public IP tắt.
- Target Group hiển thị Healthy.
- CloudWatch Logs nhận stdout/stderr.

## Lỗi thường gặp

| Lỗi | Cách xử lý |
|---|---|
| `CannotPullContainerError` | Kiểm tra ECR URI, execution role, NAT route và image architecture |
| Target unhealthy | Kiểm tra `/`, port 3000, SG và bind address |
| Task dừng ngay | Mở stopped reason và CloudWatch log |
| Secret lộ | Rotate secret và chuyển sang SSM/Secrets Manager |

## Tóm tắt

Backend được triển khai theo mô hình immutable container: build trên EC2, lưu ở ECR và chạy bằng ECS Fargate.



- Phần trước: [5.3. Xây dựng hạ tầng mạng](../5.3-Network-infrastructure/)
- Phần tiếp theo: [5.4.1. Tạo ECR](./5.4.1-Create-ecr/)
