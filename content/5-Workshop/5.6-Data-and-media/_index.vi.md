---
title: "Database và media"
date: 2026-07-19
weight: 6
chapter: false
pre: " <b> 5.6. </b> "
---



## Mục tiêu

Cấu hình MongoDB Atlas bên ngoài AWS VPC và chuyển đổi cơ chế media từ Cloudinary/local `/uploads` sang S3 Media theo phạm vi workshop.

## Kiến trúc dữ liệu

```text
ECS Fargate → Private Route Table → NAT Gateway/EIP → IGW → MongoDB Atlas
ECS Fargate --Task Role--> S3 Media Bucket
```

<!-- CHÈN ẢNH TẠI ĐÂY:
Vẽ MongoDB Atlas ngoài VPC, NAT EIP trong Atlas Access List và S3 Media truy cập bằng ECS Task Role.
Thông tin cần che: Account ID, ARN, địa chỉ IP công khai, token, cookie, secret và dữ liệu người dùng nếu xuất hiện.
-->

![Data and media overview](/images/5-Workshop/5.6-Data-and-media/data-and-media-overview.png)

## Các bước thực hiện

1. [5.6.1. Cấu hình MongoDB Atlas](./5.6.1-Configure-mongodb-atlas/)
2. [5.6.2. Tạo S3 Media Bucket và điều chỉnh backend](./5.6.2-Create-media-bucket/)

### Lưu ý

Mã nguồn hiện tại tạo thư mục local `uploads` và hỗ trợ Cloudinary. Vì vậy, không được viết rằng ứng dụng đã tích hợp S3 đầy đủ. Phần S3 yêu cầu thay đổi code, biến môi trường, IAM và kiểm thử.

## Kiểm tra kết quả

- Atlas IP Access List chỉ chứa NAT EIP và IP quản trị cần thiết.
- Không dùng `0.0.0.0/0` cho cấu hình cuối.
- `MONGO_URI` được inject qua cơ chế secret chưa xác nhận.
- Media upload thực sự xuất hiện trong S3 trước khi đánh dấu hoàn thành.

## Lỗi thường gặp

| Lỗi | Cách xử lý |
|---|---|
| Atlas timeout | Kiểm tra NAT route, EIP access list và SG egress |
| Media mất khi task restart | Hoàn tất chuyển local upload sang S3 hoặc dùng Cloudinary |
| S3 AccessDenied | Kiểm tra Task Role, bucket policy và object key |

## Tóm tắt

Database và media đã được tách khỏi filesystem tạm thời của container.



- Phần trước: [5.5. Triển khai frontend](../5.5-Frontend-deployment/)
- Phần tiếp theo: [5.6.1. Cấu hình MongoDB Atlas](./5.6.1-Configure-mongodb-atlas/)
