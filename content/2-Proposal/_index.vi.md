---
title: "Bản đề xuất"
date: 2024-01-01
weight: 2
chapter: false
pre: " <b> 2. </b> "
---

# SportWear E-Commerce Platform
## Website bán quần áo thể thao triển khai trên AWS

### 1. Tóm tắt điều hành

**SportWear** là website bán quần áo thể thao, cho phép người dùng xem sản phẩm, thêm vào giỏ hàng, đặt hàng và upload hình ảnh sản phẩm.

Hệ thống triển khai trên AWS với tách biệt rõ **frontend** (S3 + CloudFront) và **backend** (ECS Fargate phía sau Application Load Balancer). Người dùng resolve domain qua **Route 53**. **AWS WAF** được gắn với phân phối **CloudFront** để bảo vệ web cơ bản, còn CloudFront tăng tốc và cache frontend. Thông tin nhạy cảm lưu trong **Secrets Manager**. Docker image lưu trên **Amazon ECR**. Cơ sở dữ liệu dùng dịch vụ managed (**Amazon DocumentDB**, tương thích MongoDB). Các dịch vụ hỗ trợ kết nối riêng qua **VPC endpoint** (Interface/PrivateLink cho ECR, Secrets Manager, CloudWatch; Gateway endpoint cho S3).

### 2. Tuyên bố vấn đề

**Vấn đề**
- Cửa hàng quần áo thể thao nhỏ thường thiếu website bán hàng có thể mở rộng cho catalog, giỏ hàng, đặt hàng và quản lý ảnh.
- Host toàn bộ trên một server public khó bảo mật, scale và giám sát.

**Giải pháp**
- Xây dựng SportWear dạng cloud-native trên AWS.
- Frontend tĩnh trên **S3**, phân phối bằng **CloudFront**.
- API động trên **ECS Fargate** trong **Private Subnet**, phía trước là **ALB**.
- Database managed với **DocumentDB**.
- High availability với **2 Availability Zones**, mỗi AZ có **Public Subnet** và **Private Subnet**.
- Truy cập dịch vụ AWS qua **VPC endpoint** (PrivateLink cho ECR / Secrets Manager / CloudWatch; Gateway cho S3).

**Lợi ích**
- Tải trang nhanh nhờ CDN.
- An toàn hơn: backend không có public IP.
- Vận hành dễ hơn với managed DB, Secrets Manager và CloudWatch.
- Kiến trúc bám theo góp ý mentor (high-level, Multi-AZ, VPC endpoint, managed database).

### 3. Kiến trúc giải pháp

#### Luồng request
1. Người dùng truy cập domain qua **Amazon Route 53**.
2. **Amazon CloudFront** (gắn **AWS WAF** web ACL) phân phối frontend từ **S3 Frontend Bucket**.
3. Chức năng động (xem sản phẩm, giỏ hàng, đặt hàng, upload ảnh) được chuyển từ CloudFront tới **Application Load Balancer**.
4. ALB chuyển tiếp tới **ECS Fargate** trong **Private Subnet**.
5. Backend xử lý API: **product**, **cart**, **order**, **user**, **upload image**.
6. DocumentDB, secret và ảnh được truy cập từ private subnet; logs/metrics đưa về **CloudWatch**.

#### Kiến trúc high-level (đã chỉnh theo mentor)

![SportWear High-Level Architecture](/images/2-Proposal/sportwear_architecture.png)

**Ghi chú thiết kế (high-level)**
- Sơ đồ tập trung thành phần chính (không đi sâu Security Group / OAC).
- **2 AZ**, mỗi AZ có **Public Subnet** + **Private Subnet**.
- **VPC endpoint**: Interface/PrivateLink cho **Secrets Manager**, **CloudWatch**, **ECR**; Gateway endpoint cho **S3**.
- Database: **Amazon DocumentDB** (managed, tương thích MongoDB). Phương án thay thế: **Amazon DynamoDB**.

#### Dịch vụ AWS sử dụng
| Lớp | Dịch vụ | Vai trò |
| --- | --- | --- |
| DNS | Amazon Route 53 | Định tuyến domain |
| Bảo mật edge | AWS WAF | Lọc request web |
| CDN | Amazon CloudFront | Cache & phân phối frontend |
| Lưu frontend | Amazon S3 | Static website assets |
| Cân bằng tải | Application Load Balancer | Phân phối traffic API |
| Compute | Amazon ECS Fargate | Chạy container backend |
| Registry | Amazon ECR | Lưu Docker image |
| Secrets | AWS Secrets Manager | DB URL, JWT secret, API key |
| Giám sát | Amazon CloudWatch | Logs & metrics |
| Kết nối riêng | VPC endpoint | PrivateLink (ECR, Secrets Manager, CloudWatch) + S3 Gateway |
| Database | Amazon DocumentDB | Database managed tương thích MongoDB |
| Mạng | VPC (2 AZ) | Public + Private subnet cho HA |

### 4. Triển khai kỹ thuật

**Các giai đoạn**
1. Nền tảng: IAM, VPC, EC2/S3 cơ bản, CLI, giám sát.
2. Thiết kế kiến trúc: yêu cầu SportWear, chọn dịch vụ, VPC Multi-AZ.
3. Triển khai frontend: S3 + CloudFront.
4. Triển khai backend: Docker → ECR → ECS Fargate + ALB.
5. Bảo mật & vận hành: WAF, Secrets Manager, CloudWatch, kết nối DB, test API.
6. Tối ưu: góp ý mentor, Multi-AZ, VPC endpoint, checklist HA & tài liệu.

**API chính**
- Product, Cart, Order, User, Upload image

### 5. Lộ trình & Mốc triển khai

| Giai đoạn | Nội dung |
| --- | --- |
| Tuần 1–6 | Nền tảng AWS (Console, EC2, S3, CloudFront, CloudWatch, CLI) |
| Tuần 7 | Phân tích yêu cầu & thiết kế high-level SportWear |
| Tuần 8 | Frontend (S3/CloudFront) + Backend (ECS/ECR/ALB) |
| Tuần 9 | WAF, Secrets Manager, CloudWatch, DB & kiểm thử API |
| Tuần 10 | Tối ưu Multi-AZ & VPC endpoint |
| Tuần 11 | Kiến trúc kiểu production, test e-commerce, tài liệu & hoàn thiện báo cáo (24/07 – 31/07) |

### 6. Ước tính ngân sách

Mục tiêu: giữ chi phí lab thấp — dùng **Free Tier / credit sinh viên** cho dịch vụ đủ điều kiện, và theo dõi kỹ dịch vụ trả phí (đặc biệt DocumentDB).

Các hạng mục cần theo dõi chi phí:
- CloudFront data transfer
- ECS Fargate task hours
- DocumentDB instance hours (thường ngoài Free Tier)
- S3 storage & requests
- ALB hours

Dùng [AWS Pricing Calculator](https://calculator.aws/) trước khi scale và bật billing alert.

### 7. Đánh giá rủi ro

| Rủi ro | Mức ảnh hưởng | Giảm thiểu |
| --- | --- | --- |
| Chi phí AWS phát sinh | Trung bình | Ưu tiên Free Tier khi được, billing alarm, dọn tài nguyên |
| Sự cố AZ / dịch vụ | Trung bình | VPC Multi-AZ + ALB trải AZ |
| Lộ secret | Cao | Secrets Manager + IAM least privilege |
| Lỗi API / DB | Trung bình | CloudWatch logs/metrics + health check |
| Thay đổi sau review kiến trúc | Thấp | Giữ sơ đồ high-level; chỉnh theo mentor |

### 8. Kết quả kỳ vọng

- Website SportWear chạy trên AWS: xem sản phẩm, giỏ hàng, đặt hàng, upload ảnh.
- Kiến trúc high-level rõ ràng với Multi-AZ và VPC endpoint.
- Backend nằm private subnet; frontend tăng tốc bằng CloudFront.
- Database managed (DocumentDB), secrets và giám sát tập trung.
- Hồ sơ báo cáo: proposal, worklog, tài liệu workshop.
