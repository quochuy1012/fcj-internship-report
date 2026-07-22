---
title: "Tổng quan workshop"
date: 2026-07-19
weight: 1
chapter: false
pre: " <b> 5.1. </b> "
---



## Mục tiêu

Giới thiệu dự án, công nghệ, kiến trúc mục tiêu và phạm vi workshop; đồng thời phân biệt rõ máy EC2 dùng để build với ECS Fargate dùng để chạy backend production.

## Giới thiệu BravelSport

BravelSport là nền tảng thương mại điện tử và đặt sân thể thao tại `https://bravelsport.com/`. Hệ thống hỗ trợ các chức năng như duyệt sản phẩm, giỏ hàng, đặt hàng, đặt sân, quản lý tài khoản và chat realtime.

Mã nguồn tham khảo: `https://github.com/huu-luan-2004/ShopBanDoAo`.

Công nghệ chính:

- Frontend: React, Vite, TypeScript.
- Backend: NestJS, TypeScript.
- Database: MongoDB thông qua Mongoose.
- Authentication: JWT.
- Realtime: Socket.IO.
- REST API path: `/api/*`.
- Socket.IO transport path: `/socket.io/*`.
- Socket.IO namespace: `/chat`.
- Backend port mặc định: `3000`.
- Health check path ban đầu: `/`.
- Đóng gói backend: Docker.



![Trang chủ BravelSport](/fcj-internship-report/images/5-Workshop/5.1-Workshop-overview/bravelsport-homepage.jpg)

## Mục tiêu workshop

Sau workshop, người thực hiện có thể:

1. Thiết kế VPC gồm Public Subnet và Private Subnet trên hai Availability Zone.
2. Dùng EC2 Ubuntu làm máy build và triển khai.
3. Build backend Docker image, push lên ECR và chạy bằng ECS Fargate.
4. Đưa frontend React/Vite lên S3 private và phân phối bằng CloudFront OAC.
5. Định tuyến `/api/*` và `/socket.io/*` từ CloudFront đến ALB.
6. Kết nối ECS với MongoDB Atlas qua NAT Gateway.
7. Gửi log container đến CloudWatch Logs.
8. Mở rộng cơ chế media từ Cloudinary/local `/uploads` sang S3 Media.
9. Kiểm thử, đánh giá bảo mật, theo dõi chi phí và xóa tài nguyên.

## Kiến trúc ba lớp

| Lớp | Thành phần | Vai trò |
|---|---|---|
| Presentation | Route 53, CloudFront, WAF, S3 Frontend | DNS, TLS, phân phối static frontend và lọc request |
| Application | ALB, Target Group `ip`, ECS Fargate | Chạy REST API và Socket.IO trên cổng `3000` |
| Data | MongoDB Atlas, S3 Media | Lưu dữ liệu nghiệp vụ và media |

## Vai trò EC2 Build Machine

EC2 Ubuntu nằm trong khung **Development & Deployment**. Máy này chỉ được dùng để clone source, cài công cụ, build frontend, build Docker image, đăng nhập ECR, push image và chạy AWS CLI.

Luồng deployment:

```text
GitHub
  └─→ EC2 Ubuntu Build Machine
        ├─→ build frontend → S3 Frontend Bucket
        └─→ docker build → Amazon ECR → ECS Fargate pull image
```

EC2 không nhận request từ người dùng, không nối với ALB và không chạy backend production.



![Luồng build và triển khai](/fcj-internship-report/images/5-Workshop/5.1-Workshop-overview/deployment-flow.jpg)

## Vai trò ECS Fargate

ECS Fargate là runtime của backend NestJS. Mỗi task chạy container trên cổng `3000`, nằm trong Private Subnet, không có public IP và chỉ nhận inbound từ SG-ALB thông qua Target Group loại `ip`.

Task Execution Role được nền tảng ECS dùng để pull image và gửi log. Task Role được mã ứng dụng dùng để truy cập S3 Media hoặc các AWS service khác được cấp quyền.

## Kiến trúc tổng thể

Kiến trúc gồm các vùng logic:

- **Edge & Frontend:** Route 53, CloudFront, AWS WAF và S3 Frontend.
- **Development & Deployment:** GitHub, EC2 Ubuntu Build Machine và ECR.
- **Amazon VPC – Backend Network:** ALB trong Public Subnet; ECS Fargate trong Private Subnet; NAT Gateway, Elastic IP và Internet Gateway.
- **Supporting Services:** IAM, CloudWatch Logs và S3 Media.
- **External Database:** MongoDB Atlas nằm ngoài AWS VPC.



![BravelSport AWS Architecture](/fcj-internship-report/images/5-Workshop/architecture/bravelsport-aws-architecture.png)

## Các luồng hoạt động

### Frontend

`User → Route 53 → CloudFront + AWS WAF → S3 Frontend Bucket`

### REST API

`User → Route 53 → CloudFront → /api/* → ALB → Target Group (ip) → ECS Fargate:3000`

### Socket.IO

`User → CloudFront → /socket.io/* → ALB → ECS Fargate → namespace /chat`

`/socket.io/*` là transport path dùng để bắt tay và trao đổi frame; `/chat` là namespace logic của ứng dụng.

### MongoDB Atlas

`ECS Fargate → Private Route Table → NAT Gateway → Internet Gateway → MongoDB Atlas`

Elastic IP của NAT Gateway được thêm vào Atlas IP Access List. Không dùng `0.0.0.0/0` trong cấu hình cuối.

### Media

`User → CloudFront → ALB → ECS Fargate → S3 Media Bucket`

Mã nguồn hiện có thể dùng Cloudinary hoặc local `/uploads`; S3 Media là thay đổi hoặc mở rộng được thực hiện trong workshop.

### Build và deployment

`GitHub → EC2 Build Machine → ECR → ECS Fargate`

Nhánh frontend: `EC2 Build Machine → S3 Frontend Bucket`.

## Các bước thực hiện

1. Chuẩn bị tài khoản, EC2 build machine, domain, certificate và source code.
2. Xây dựng VPC, subnet, route và security group.
3. Tạo ECR, IAM roles, CloudWatch Log Group, ALB, Target Group và ECS Service.
4. Build frontend, upload S3, tạo CloudFront, WAF và Route 53 Alias.
5. Cấu hình MongoDB Atlas và mở rộng media sang S3.
6. Kiểm thử end-to-end, rà soát bảo mật và clean-up.

## Kiểm tra kết quả

- Diagram không có đường `ALB → EC2`.
- User không truy cập trực tiếp EC2, NAT Gateway, Internet Gateway hoặc ECS.
- CloudFront có hai behavior riêng cho `/api/*` và `/socket.io/*`.
- Target Group dùng loại `ip`.
- MongoDB Atlas nằm ngoài VPC.
- IAM được thể hiện bằng nét đứt, không phải data flow.

## Lỗi thường gặp

| Lỗi | Cách sửa |
|---|---|
| Vẽ EC2 nằm giữa ALB và ECS | Đưa EC2 sang khung Development & Deployment |
| Gộp `/socket.io/*` và `/chat` | Ghi rõ transport path và namespace là hai khái niệm khác nhau |
| Khẳng định S3 Media đã có sẵn | Ghi đây là phần mở rộng workshop |
| Vẽ CloudWatch/ECR qua một “public endpoints” chung | Nối trực tiếp ECS đến từng dịch vụ hoặc ghi rõ đây là association/outbound |

## Tóm tắt

Workshop sử dụng EC2 để build, ECR để lưu image và ECS Fargate để chạy backend. Runtime traffic đi qua CloudFront, WAF và ALB; database nằm ngoài VPC; media được chuyển đổi dần sang S3.

## Danh sách ảnh

| STT | Tên file | Nội dung ảnh | Vị trí chèn |
|---:|---|---|---|
| 1 | `bravelsport-homepage.jpg` | Trang chủ BravelSport qua domain của dự án | Sau phần giới thiệu dự án |
| 2 | `deployment-flow.jpg` | Luồng GitHub → EC2 Build Machine → ECR → ECS Fargate | Sau phần vai trò EC2 Build Machine |
| 3 | `bravelsport-aws-architecture.png` | Sơ đồ kiến trúc AWS tổng thể | Sau phần kiến trúc tổng thể |

## Điều hướng

- Phần trước: [Workshop](../)
- Phần tiếp theo: [5.2. Các bước chuẩn bị](../5.2-Prerequisite/)
