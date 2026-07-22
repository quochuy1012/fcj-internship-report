---
title: "Tạo S3 Media Bucket và điều chỉnh backend"
date: 2026-07-19
weight: 2
chapter: false
pre: " <b> 5.6.2. </b> "
---


## Mục tiêu

Thay thế cơ chế lưu trữ media trên local filesystem hoặc Cloudinary bằng Amazon S3 để đảm bảo dữ liệu bền vững khi triển khai trên Amazon ECS Fargate.

## Hiện trạng mã nguồn

Backend hiện tạo thư mục `uploads` và phục vụ URL `/uploads/`; file cấu hình mẫu cũng hỗ trợ Cloudinary. Local filesystem của Fargate không phù hợp để lưu media bền vững qua deployment. Do đó workshop sử dụng Amazon S3 làm Media Storage.

## Các bước thực hiện

### 1. Tạo S3 Media Bucket

Tạo bucket với các thông số:

| Thông số | Giá trị |
|---|---|
| Bucket name | `bravel-uploads` |
| Region | `ap-southeast-1 (Singapore)` |
| Block Public Access | Enabled |
| Default Encryption | Enabled (SSE-S3) |
| Versioning | Enabled (khuyến nghị) |
| Lifecycle | Cấu hình theo nhu cầu lưu trữ |

Bucket này chỉ dùng để lưu media của ứng dụng và không dùng chung với S3 Frontend Bucket.

<!-- CHÈN ẢNH TẠI ĐÂY:
Chụp danh sách bucket hoặc trang bucket bravel-uploads.
Ảnh cần hiển thị:
- Bucket name: bravel-uploads
- Region: ap-southeast-1

Che Account ID nếu xuất hiện.
-->

![Create S3 media bucket](/images/5-Workshop/5.6-Data-and-media/create-media-bucket.jpg)

### 2. Cấp quyền truy cập S3

Tạo IAM Policy:

```text
BackendUploadS3Policy
```

Policy này cấp quyền cho ECS Task Role:

- `s3:GetObject`
- `s3:PutObject`
- `s3:DeleteObject`
- `s3:ListBucket`

Quyền chỉ được giới hạn trên bucket:

```text
bravel-uploads
```

Không sử dụng `AmazonS3FullAccess` hoặc `AdministratorAccess`.

### 3. Gắn quyền Task Role

Giới hạn policy ở bucket và prefix media cần thiết.

<!-- CHÈN ẢNH TẠI ĐÂY:
Chụp policy chỉ định đúng bucket/prefix. Che ARN/Account ID và không hiển thị policy khác.
Thông tin cần che: Account ID, ARN, địa chỉ IP công khai, token, cookie, secret và dữ liệu người dùng nếu xuất hiện.
-->

![Media Task Role policy](/images/5-Workshop/5.6-Data-and-media/media-task-role-policy.jpg)

### 4. Triển khai revision mới

- Build image mới trên EC2.
- Push ECR.
- Tạo Task Definition revision.
- Update ECS Service.
- Theo dõi deployment và log.

### 5. Kiểm thử upload

- Upload ảnh qua giao diện/API.
- Xác nhận object xuất hiện trong S3.
- Xác nhận URL media hiển thị trên BravelSport.
- Kiểm tra task restart không làm mất media.

<!-- CHÈN ẢNH TẠI ĐÂY:
Chụp object trong S3 và ảnh hiển thị trên BravelSport. Che tên người dùng, metadata và URL signed có token.
Thông tin cần che: Account ID, ARN, địa chỉ IP công khai, token, cookie, secret và dữ liệu người dùng nếu xuất hiện.
-->

![Verify media upload](/images/5-Workshop/5.6-Data-and-media/verify-media-upload.jpg)

## Kiểm tra kết quả

- S3 bucket private.
- App upload bằng Task Role.
- Không có AWS access key trong Task Definition.
- Media vẫn tồn tại sau khi task bị thay thế.

## Lỗi thường gặp

| Lỗi | Cách xử lý |
|---|---|
| AccessDenied | Kiểm tra Task Role resource ARN và prefix |
| Ảnh không hiển thị | Xác nhận URL strategy, CloudFront/media delivery và CORS |
| Upload local thay vì S3 | Kiểm tra code path và environment flag |
| Public bucket | Giữ bucket private; dùng CloudFront/signed URL nếu cần |

## Tóm tắt

Media đã được chuyển sang object storage bền vững hoặc được đánh dấu rõ là chưa hoàn thành nếu chưa có code và bằng chứng thật.


- Phần trước: [5.6.1. MongoDB Atlas](../5.6.1-Configure-mongodb-atlas/)
- Phần tiếp theo: [5.7. Kiểm thử end-to-end](../../5.7-Testing/)
