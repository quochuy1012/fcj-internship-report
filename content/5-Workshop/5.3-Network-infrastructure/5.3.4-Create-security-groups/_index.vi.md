---
title: "Tạo Security Group"
date: 2026-07-19
weight: 4
chapter: false
pre: " <b> 5.3.4. </b> "
---


## Mục tiêu

Giới hạn lưu lượng mạng giữa Application Load Balancer, ECS Fargate và EC2 Build Machine, đảm bảo chỉ các thành phần cần thiết mới được phép giao tiếp với nhau theo kiến trúc của hệ thống.

## Các bước thực hiện

### 1. Tạo Security Group cho Application Load Balancer

Tạo Security Group:

```text
web-ban-ao-alb-sg
```

Gắn Security Group này cho Application Load Balancer.

#### Inbound Rules

| Type | Protocol | Port | Source |
|---|---|---:|---|
| HTTP | TCP | 80 | 0.0.0.0/0 |
| HTTPS | TCP | 443 | 0.0.0.0/0 |

- HTTP dùng để chuyển hướng sang HTTPS nếu cấu hình.
- HTTPS là cổng chính tiếp nhận request từ Internet.

#### Outbound Rules

| Type | Protocol | Port | Destination |
|---|---|---:|---|
| Custom TCP | TCP | 3000 | `bravelsport-sg-ecs` |

Application Load Balancer chỉ chuyển request đến ECS Fargate thông qua cổng `3000`.

---

### 2. Tạo Security Group cho ECS Fargate

Tạo Security Group:

```text
bravelsport-sg-ecs
```

Gắn Security Group này cho ECS Service.

#### Inbound Rules

| Type | Protocol | Port | Source |
|---|---|---:|---|
| Custom TCP | TCP | 3000 | `web-ban-ao-alb-sg` |

Chỉ Application Load Balancer được phép truy cập backend.

Không mở:

```text
TCP 3000 từ 0.0.0.0/0
```

ECS Task không có Public IP và không nhận request trực tiếp từ Internet.

#### Outbound Rules

Cho phép ECS Task truy cập:

- Amazon ECR
- Amazon CloudWatch Logs
- Amazon S3
- AWS Secrets Manager
- MongoDB Atlas

Trong workshop có thể giữ outbound mặc định:

| Type | Protocol | Port | Destination |
|---|---|---:|---|
| All traffic | All | All | 0.0.0.0/0 |

Lưu lượng outbound của ECS sẽ đi:

```text
ECS Fargate
→ NAT Gateway
→ Internet Gateway
→ AWS Services / MongoDB Atlas
```

---

### 3. Tạo Security Group cho EC2 Build Machine

Tạo Security Group:

```text
bravelsport-sg-build
```

EC2 Build Machine chỉ phục vụ:

- Clone source code.
- Build frontend React/Vite.
- Build Docker Image.
- Push Image lên Amazon ECR.
- Đồng bộ frontend lên Amazon S3.
- Triển khai ứng dụng bằng AWS CLI.

EC2 Build Machine không tham gia xử lý request của người dùng.

#### Inbound Rules

| Type | Protocol | Port | Source |
|---|---|---:|---|
| SSH | TCP | 22 | `<ADMIN_PUBLIC_IP>/32` |

Chỉ mở SSH từ địa chỉ IP quản trị.

Không mở:

- TCP 3000
- HTTP 80
- HTTPS 443

Nếu sử dụng AWS Systems Manager Session Manager thì không cần mở SSH.

#### Outbound Rules

| Type | Protocol | Port | Destination |
|---|---|---:|---|
| HTTPS | TCP | 443 | 0.0.0.0/0 |

EC2 sử dụng HTTPS để truy cập:

- GitHub
- Amazon ECR
- Amazon S3
- AWS APIs
- npm Registry
- Docker Registry

<!-- CHÈN ẢNH TẠI ĐÂY:
Chụp Security Group của:

- web-ban-ao-alb-sg
- bravelsport-sg-ecs
- bravelsport-sg-build

Ảnh cần hiển thị:

- HTTP 80
- HTTPS 443
- TCP 3000
- SSH 22

Che:

- AWS Account ID
- Security Group ID
- VPC ID
- Public IP quản trị
-->

![Security Group BravelSport](/images/5-Workshop/5.3-Network-infrastructure/create-security-groups.png)

## Kiểm tra kết quả

- Application Load Balancer chỉ mở HTTP `80` và HTTPS `443`.
- ECS Fargate chỉ cho phép inbound TCP `3000` từ Security Group của ALB.
- ECS Task không có Public IP.
- EC2 Build Machine chỉ mở SSH từ IP quản trị.
- EC2 không mở cổng `3000`.
- Application Load Balancer không chuyển request đến EC2 Build Machine.
- ECS Task có thể truy cập Amazon ECR, CloudWatch Logs, Amazon S3, AWS Secrets Manager và MongoDB Atlas thông qua NAT Gateway.

## Lỗi thường gặp

| Lỗi | Cách xử lý |
|---|---|
| Target unhealthy do timeout | Kiểm tra SG-ALB outbound và SG-ECS inbound port `3000` |
| ALB truy cập được trực tiếp | Giới hạn truy cập bằng CloudFront origin-facing managed prefix list hoặc custom origin header |
| ECS không kết nối được MongoDB Atlas | Kiểm tra outbound rule của SG-ECS và IP Access List của MongoDB Atlas |
| ECS không pull được Docker Image | Kiểm tra outbound đến Amazon ECR và NAT Gateway |
| Không ghi được CloudWatch Logs | Kiểm tra outbound đến CloudWatch Logs và IAM Execution Role |
| SSH mở toàn Internet | Thu hẹp về IP quản trị hoặc sử dụng AWS Systems Manager Session Manager |

## Tóm tắt

Security Group đã được cấu hình theo nguyên tắc **Least Privilege**, tách biệt quyền truy cập giữa Application Load Balancer, ECS Fargate và EC2 Build Machine. Backend chỉ nhận request từ Application Load Balancer, trong khi EC2 Build Machine chỉ phục vụ quá trình build và triển khai ứng dụng.


- Phần trước: [5.3.3. Cấu hình routing](../5.3.3-Configure-routing/)
- Phần tiếp theo: [5.4. Triển khai backend](../../5.4-Backend-deployment/)