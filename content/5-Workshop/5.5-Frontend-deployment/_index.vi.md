---
title: "Triển khai frontend React/Vite"
date: 2026-07-19
weight: 5
chapter: false
pre: " <b> 5.5. </b> "
---



## Mục tiêu

Build frontend trên EC2, upload `dist/` lên S3 private và cấu hình CloudFront làm điểm vào thống nhất cho frontend, REST API và Socket.IO.

## Kiến trúc delivery

```text
User → Route 53 → CloudFront + WAF
                     ├─ /* → S3 Frontend Bucket
                     ├─ /api/* → ALB
                     └─ /socket.io/* → ALB
```

<!-- CHÈN ẢNH TẠI ĐÂY:
Vẽ Route 53 → CloudFront/WAF với S3 origin và ALB origin. Ghi rõ behavior /api/* và /socket.io/*.
Thông tin cần che: Account ID, ARN, địa chỉ IP công khai, token, cookie, secret và dữ liệu người dùng nếu xuất hiện.
-->

![Frontend delivery overview](/images/5-Workshop/5.5-Frontend-deployment/frontend-delivery-overview.png)

## Các bước thực hiện

1. [5.5.1. Build frontend React/Vite](./5.5.1-Build-frontend/)
2. [5.5.2. Tạo S3 Frontend Bucket](./5.5.2-Create-s3-frontend/)
3. [5.5.3. Tạo CloudFront Distribution](./5.5.3-Create-cloudfront/)
4. [5.5.4. Cấu hình AWS WAF](./5.5.4-Configure-waf/)
5. [5.5.5. Cấu hình Route 53](./5.5.5-Configure-route53/)

## Kiểm tra kết quả

- S3 Frontend private và chỉ CloudFront OAC đọc được.
- CloudFront default behavior phục vụ SPA.
- `/api/*` và `/socket.io/*` không cache nội dung động.
- Authorization header/query/cookie cần thiết được forward theo source code.
- Route 53 Alias trỏ đến CloudFront, không trỏ trực tiếp ALB/ECS.

## Lỗi thường gặp

| Lỗi | Cách xử lý |
|---|---|
| SPA reload 403/404 | Cấu hình custom error response hoặc CloudFront Function fallback phù hợp |
| API bị cache | Dùng caching disabled policy cho `/api/*` |
| Socket.IO handshake lỗi | Kiểm tra `/socket.io/*`, query string, headers, methods và timeout |
| S3 public | Bật Block Public Access và dùng OAC |

## Tóm tắt

Frontend và backend được xuất bản qua một CloudFront distribution có WAF bảo vệ.



- Phần trước: [5.4. Triển khai backend](../5.4-Backend-deployment/)
- Phần tiếp theo: [5.5.1. Build frontend](./5.5.1-Build-frontend/)
