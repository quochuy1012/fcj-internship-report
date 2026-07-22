---
title: "Tạo IAM Role cho ECS"
date: 2026-07-19
weight: 3
chapter: false
pre: " <b> 5.4.3. </b> "
---



## Mục tiêu

Tách quyền của nền tảng ECS khỏi quyền của mã ứng dụng.

## Các bước thực hiện
### 1. ECS Task Execution Role

Trust entity: `ecs-tasks.amazonaws.com`.

Quyền chính:

- Pull Docker image từ Amazon ECR.
- Tạo log stream và gửi container logs đến Amazon CloudWatch Logs.
- Đọc secret và cấu hình từ AWS Secrets Manager khi ECS Task Definition tham chiếu các secret.

Role được gắn managed policy:

- `AmazonECSTaskExecutionRolePolicy`

Ngoài ra, role được bổ sung policy:

- `SecretsManagerAccess`

Policy này cho phép ECS lấy các secret như `MONGO_URI`, `JWT_SECRET` và các giá trị nhạy cảm khác từ AWS Secrets Manager khi task khởi động.

### 2. ECS Task Role

Trust entity: `ecs-tasks.amazonaws.com`.

Quyền ứng dụng:

- `s3:GetObject`, `s3:PutObject`, `s3:DeleteObject` trên các object trong S3 Media Bucket của BravelSport.
- `s3:ListBucket` trên S3 Media Bucket nếu ứng dụng cần liệt kê object.
- Chỉ bổ sung các quyền AWS API khác khi mã nguồn thực sự sử dụng.

Quyền phải được giới hạn theo đúng ARN của S3 Media Bucket, không áp dụng cho toàn bộ tài nguyên S3 trong tài khoản.

Không gắn `AdministratorAccess` hoặc `AmazonS3FullAccess`.

### 3. Secret và IAM

IAM không lưu:

- `MONGO_URI`.
- `JWT_SECRET`.
- Google client secret.
- VNPay hash secret.
- Cloudinary API secret.
- SMTP password.

Cơ chế lưu secret: `[TO BE CONFIRMED: SSM Parameter Store hoặc AWS Secrets Manager]`.

<!-- CHÈN ẢNH TẠI ĐÂY:
Chụp danh sách hoặc trang chi tiết hai role. Che ARN và Account ID đầy đủ; không mở tab hiển thị secret value.
Thông tin cần che: Account ID, ARN, địa chỉ IP công khai, token, cookie, secret và dữ liệu người dùng nếu xuất hiện.
-->

![Tạo ECS IAM roles - bước 1](/images/5-Workshop/5.4-Backend-deployment/create-iam-roles1.png)

![Tạo ECS IAM roles - bước 2](/images/5-Workshop/5.4-Backend-deployment/create-iam-roles2.png)

## Kiểm tra kết quả

- Task Execution Role và Task Role là hai role riêng.
- Trust policy dùng `ecs-tasks.amazonaws.com`.
- Task Role chỉ có quyền S3 Media cần thiết.
- Không có secret trong IAM policy.

## Lỗi thường gặp

| Lỗi | Cách xử lý |
|---|---|
| App không truy cập S3 | Gắn policy vào Task Role, không phải Execution Role |
| ECS không pull image | Kiểm tra Execution Role và ECR permissions |
| Người tạo service không pass được role | Cấp `iam:PassRole` giới hạn cho role triển khai |
| Dùng IAM để “lưu secret” | Chuyển value sang SSM/Secrets Manager |

## Tóm tắt

Hai role đã được phân tách theo đúng trách nhiệm của ECS agent và ứng dụng.



- Phần trước: [5.4.2. Build và push image](../5.4.2-Build-and-push-image/)
- Phần tiếp theo: [5.4.4. Tạo ALB và Target Group](../5.4.4-Create-alb/)
