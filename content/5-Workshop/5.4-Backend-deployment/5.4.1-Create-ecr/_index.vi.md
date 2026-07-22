---
title: "Tạo Amazon ECR Repository"
date: 2026-07-19
weight: 1
chapter: false
pre: " <b> 5.4.1. </b> "
---


## Mục tiêu

Tạo private repository lưu Docker image backend BravelSport.

## Các bước thực hiện

1. Mở Amazon ECR tại Region.
2. Chọn **Create repository**.
3. Visibility: **Private**.
4. Repository name đề xuất: `web-ban-ao-backend`.
5. Bật **Image tag immutability** ở chế độ `Immutable` để ngăn ghi đè Docker image đã tồn tại. Mỗi lần triển khai sử dụng một tag duy nhất, ví dụ `v1.0.0`, `build-001` hoặc Git commit SHA.

6. Bật **Basic scanning – Scan on push** để Amazon ECR tự động quét lỗ hổng hệ điều hành mỗi khi Docker image mới được đẩy lên repository.
7. Tạo lifecycle policy để xóa image cũ nếu workshop kéo dài.



![ECR repository](/images/5-Workshop/5.4-Backend-deployment/create-ecr.png)

## Kiểm tra kết quả

- Repository tồn tại đúng Region.
- Repository là private.
- EC2 IAM Role có quyền push tối thiểu.

## Lỗi thường gặp

| Lỗi | Cách xử lý |
|---|---|
| Repository trùng tên | Dùng repository hiện có sau khi xác nhận đúng project |
| AccessDenied | Bổ sung quyền ECR tối thiểu cho EC2 instance profile |
| Sai Region | Đăng nhập và push đúng registry Region |

## Tóm tắt

ECR sẵn sàng nhận Docker image backend.



- Phần trước: [5.4. Triển khai backend](../)
- Phần tiếp theo: [5.4.2. Build và push image](../5.4.2-Build-and-push-image/)
