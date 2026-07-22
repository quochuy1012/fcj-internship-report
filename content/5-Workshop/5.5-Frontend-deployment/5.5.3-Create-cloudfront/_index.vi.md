---
title: "Tạo CloudFront Distribution"
date: 2026-07-19
weight: 3
chapter: false
pre: " <b> 5.5.3. </b> "
---



## Mục tiêu

Cấu hình Amazon CloudFront để sử dụng S3 Private Bucket làm origin cho frontend và Application Load Balancer làm origin cho REST API và Socket.IO.

## Các bước thực hiện

### 1. Tạo S3 Origin và Origin Access Control (OAC)

1. Tạo CloudFront Distribution.
2. Chọn bucket `myapp-frontend-sport` làm S3 Origin.
3. Tạo **Origin Access Control (OAC)** với tùy chọn **Sign requests (recommended)**.
4. Cập nhật Bucket Policy để chỉ cho phép CloudFront Distribution đọc object trong bucket.

Bucket Policy sẽ có dạng:

```json
{
  "Version": "2008-10-17",
  "Statement": [
    {
      "Sid": "AllowCloudFrontServicePrincipal",
      "Effect": "Allow",
      "Principal": {
        "Service": "cloudfront.amazonaws.com"
      },
      "Action": "s3:GetObject",
      "Resource": "arn:aws:s3:::myapp-frontend-sport/*",
      "Condition": {
        "StringEquals": {
          "AWS:SourceArn": "arn:aws:cloudfront::<ACCOUNT_ID>:distribution/<DISTRIBUTION_ID>"
        }
      }
    }
  ]
}
```

<!-- CHÈN ẢNH TẠI ĐÂY:
Chụp Bucket Policy sau khi tạo OAC.
Che Account ID và Distribution ID.
-->

![CloudFront OAC Bucket Policy](/images/5-Workshop/5.5-Frontend-deployment/cloudfront-oac-policy.png)

### 2. Tạo ALB Origin

Thêm một Origin mới cho CloudFront với các thông số:

- Origin domain: DNS Name của Application Load Balancer.
- Origin protocol policy: **HTTP Only**.
- Origin Shield: Disabled.
- Không bật Origin Path.
- Có thể thêm Custom Header nếu muốn giới hạn truy cập trực tiếp vào ALB.

### 3. Default Behavior `/*`

Cấu hình:

- Origin: `myapp-frontend-sport`
- Viewer protocol policy: **Redirect HTTP to HTTPS**
- Allowed methods: `GET`, `HEAD`
- Compress objects automatically: Enabled
- Cache policy: **CachingOptimized**

Behavior này phục vụ toàn bộ nội dung frontend như:

- `index.html`
- CSS
- JavaScript
- Hình ảnh
- Font

### 4. Behavior `/api/*`

Cấu hình:

- Origin: Application Load Balancer
- Allowed methods:

```text
GET
HEAD
OPTIONS
PUT
POST
PATCH
DELETE
```

- Cache policy: **CachingDisabled**
- Origin request policy: **AllViewerExceptHostHeader**
- Forward:
  - Authorization
  - Content-Type
  - Query Strings
  - Cookies

Behavior này chuyển toàn bộ REST API đến backend NestJS.

### 5. Behavior `/socket.io/*`

Cấu hình:

- Origin: Application Load Balancer
- Cache policy: **CachingDisabled**
- Allowed methods:

```text
GET
HEAD
OPTIONS
POST
```

- Forward toàn bộ Query Strings.
- Forward các Header cần cho WebSocket Upgrade.
- Không tạo Behavior `/chat` vì `/chat` là Socket.IO Namespace, không phải URL Path.

### 6. React Router Fallback

Trong CloudFront Distribution, cấu hình Custom Error Responses:

| HTTP Error | Response Page | HTTP Response Code |
|---|---|---|
| 403 | `/index.html` | `200` |
| 404 | `/index.html` | `200` |

Cấu hình này giúp React Router hoạt động đúng khi người dùng tải trực tiếp các URL như:

```text
/products
/profile
/orders
```

Không áp dụng cơ chế fallback cho:

- `/api/*`
- `/socket.io/*`

<!-- CHÈN ẢNH TẠI ĐÂY:
Chụp CloudFront Distribution sau khi hoàn tất.
Ảnh cần hiển thị:
- Distribution Status: Deployed
- S3 Origin
- ALB Origin
- Behaviors:
  - /*
  - /api/*
  - /socket.io/*

Che Distribution ID, Account ID và ALB DNS Name.
-->

![CloudFront Distribution](/images/5-Workshop/5.5-Frontend-deployment/create-cloudfront.png)

## Kiểm tra kết quả

- CloudFront Distribution ở trạng thái **Deployed**.
- S3 Origin sử dụng Origin Access Control (OAC).
- Bucket Policy chỉ cho phép CloudFront đọc object.
- Default Behavior (`/*`) trỏ đến S3 Frontend Bucket.
- Behavior `/api/*` trỏ đến Application Load Balancer và tắt cache.
- Behavior `/socket.io/*` trỏ đến Application Load Balancer và hỗ trợ WebSocket.
- Người dùng không thể truy cập trực tiếp vào S3 Bucket.
- React Router hoạt động đúng khi tải lại trang.
- REST API và Socket.IO hoạt động thông qua CloudFront.

## Lỗi thường gặp

| Lỗi | Cách xử lý |
|---|---|
| API trả frontend index.html | Behavior order/path sai; đặt dynamic behaviors trước default |
| Socket.IO 400/502 | Kiểm tra query/header forwarding, origin protocol và ALB target |
| JWT mất | Forward `Authorization` bằng origin request policy |
| CloudFront 403 S3 | Sửa OAC bucket policy và origin access settings |

## Tóm tắt

CloudFront đã trở thành điểm vào chung cho frontend, API và Socket.IO.


- Phần trước: [5.5.2. Tạo S3 Frontend](../5.5.2-Create-s3-frontend/)
- Phần tiếp theo: [5.5.4. Cấu hình WAF](../5.5.4-Configure-waf/)
