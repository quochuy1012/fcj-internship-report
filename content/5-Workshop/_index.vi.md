---
title: "Workshop"
date: 2026-07-19
weight: 5
chapter: false
pre: " <b> 5. </b> "
---

# Triển khai BravelSport trên AWS

BravelSport là nền tảng thương mại điện tử và đặt sân thể thao. Mã nguồn gồm frontend React/Vite và backend NestJS; backend dùng MongoDB qua Mongoose, JWT cho xác thực và Socket.IO cho chat realtime.

Workshop này mô tả **kiến trúc mục tiêu và quy trình triển khai thực hành**. Nội dung không khẳng định website hiện tại đang chạy full kiến trúc AWS nếu chưa có bằng chứng từ AWS Console.

## Kiến trúc mục tiêu

- Frontend build thành static files, lưu S3 Frontend Bucket (private) và phân phối qua CloudFront.
- AWS WAF gắn với CloudFront để kiểm tra request tại edge.
- `/api/*` và `/socket.io/*` được CloudFront chuyển tới Application Load Balancer.
- ALB chuyển request qua Target Group loại `ip` tới ECS Fargate trên cổng `3000`.
- Socket.IO dùng path `/socket.io/*`; chat dùng namespace `/chat`.
- ECS Fargate chạy trong Private Subnet, không có public IP.
- ECS kết nối MongoDB Atlas qua NAT Gateway và Internet Gateway.
- EC2 Ubuntu chỉ là máy build/triển khai, không nhận traffic production.
- S3 Media là phần mở rộng của workshop (mã nguồn hiện có Cloudinary / local `/uploads`).

![BravelSport AWS Architecture](/images/5-Workshop/architecture/bravelsport-aws-architecture.png)

## Nội dung workshop

1. [5.1. Tổng quan workshop](./5.1-Workshop-overview/)
2. [5.2. Các bước chuẩn bị](./5.2-Prerequisite/)
3. [5.3. Xây dựng hạ tầng mạng](./5.3-Network-infrastructure/)
4. [5.4. Triển khai backend](./5.4-Backend-deployment/)
5. [5.5. Triển khai frontend](./5.5-Frontend-deployment/)
6. [5.6. Database và media](./5.6-Data-and-media/)
7. [5.7. Kiểm thử end-to-end](./5.7-Testing/)
8. [5.8. Dọn dẹp tài nguyên](./5.8-Cleanup/)

### Lưu ý

Không đưa access key, secret, MongoDB connection string, JWT secret, cookie đăng nhập hoặc Account ID đầy đủ vào tài liệu. Giá trị chưa xác nhận giữ dạng `[TO BE CONFIRMED]`.

## Kiểm tra kết quả

- Menu Hugo hiển thị đủ các phần 5.1–5.8 theo đúng thứ tự.
- Mọi đường dẫn ảnh bắt đầu bằng `/images/5-Workshop/`.
- Phân biệt rõ EC2 Build Machine và ECS Fargate runtime.

## Lỗi thường gặp

| Lỗi | Nguyên nhân | Cách xử lý |
|---|---|---|
| Menu sai thứ tự | `weight` hoặc `pre` sai | Kiểm tra front matter |
| Ảnh không hiển thị | Sai thư mục dưới `static/` | Đối chiếu `/images/5-Workshop/...` |
| Liên kết trang con lỗi | Đổi tên thư mục nhưng chưa sửa link | Kiểm tra link tương đối |

## Tóm tắt

Workshop hướng dẫn từ chuẩn bị, mạng, backend, frontend, database/media, kiểm thử đến clean-up.

## Điều hướng

- Phần tiếp theo: [5.1. Tổng quan workshop](./5.1-Workshop-overview/)
