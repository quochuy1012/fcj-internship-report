---
title: "Đề xuất dự án"
date: 2026-07-19
weight: 2
chapter: false
pre: " <b> 2. </b> "
---

# Nền tảng web thể thao BravelSport

## Đề xuất kiến trúc AWS cho hệ thống triển khai an toàn và có khả năng mở rộng

### 1. Tóm tắt dự án

BravelSport là nền tảng web thể thao được triển khai tại địa chỉ:

`https://bravelsport.com/`

Hệ thống hướng đến các nhóm người dùng như khách hàng, người chơi thể thao, câu lạc bộ, chủ sân và quản trị viên.

Các chức năng chính của BravelSport gồm:

* Xem danh sách và thông tin chi tiết sản phẩm thể thao.
* Tìm kiếm và lọc sản phẩm.
* Đăng ký và đăng nhập tài khoản.
* Quản lý giỏ hàng.
* Đặt hàng và theo dõi đơn hàng.
* Đặt sân thể thao.
* Quản lý thông tin sân và lịch đặt sân.
* Tải lên hình ảnh sản phẩm và sân thể thao.
* Quản lý người dùng, sản phẩm, đơn hàng, sân và nội dung hệ thống.

Đề xuất này trình bày phương án triển khai BravelSport trên AWS theo kiến trúc tách biệt giữa frontend, backend, hệ thống mạng, lưu trữ, giám sát và cơ sở dữ liệu.

Frontend được build thành static files, lưu trên Amazon S3 và phân phối qua Amazon CloudFront. Domain `bravelsport.com` được nhóm đăng ký và quản lý bằng Amazon Route 53. AWS WAF được gắn với CloudFront để kiểm tra request đầu vào.

Backend được đóng gói bằng Docker, lưu trên Amazon ECR và chạy bằng Amazon ECS Fargate trong private subnet. Application Load Balancer nhận request API và chuyển tiếp đến ECS task.

MongoDB Atlas được sử dụng làm cơ sở dữ liệu bên ngoài AWS. Amazon S3 lưu hình ảnh, media và tệp cấu hình của ứng dụng. AWS IAM quản lý quyền truy cập, trong khi Amazon CloudWatch thu thập logs và metrics.

Kiến trúc này phù hợp với mục tiêu workshop, MVP và hệ thống quy mô nhỏ hoặc trung bình.

---

### 2. Vấn đề cần giải quyết

BravelSport không chỉ là website tĩnh mà gồm frontend, backend API, cơ sở dữ liệu, media upload, đặt hàng, đặt sân và quản trị nội dung.

Nếu không có kiến trúc triển khai rõ ràng, hệ thống có thể gặp:

* Backend bị public trực tiếp ra Internet.
* Frontend và backend phụ thuộc cùng một máy chủ.
* Media lưu local và mất khi container thay thế.
* Khó quản lý / rollback phiên bản backend.
* Không có logs tập trung.
* Khó mở rộng khi số lượng người dùng tăng.
* Credential dễ lộ nếu lưu trong source code.
* Tài nguyên AWS tiếp tục phát sinh chi phí sau workshop.

Giải pháp đề xuất: tách frontend, backend, media và database. Frontend trên S3 + CloudFront; backend trên ECS Fargate trong private subnet; dữ liệu trên MongoDB Atlas.

---

### 3. Mục tiêu triển khai

* Triển khai BravelSport trên AWS theo kiến trúc rõ ràng.
* Tách biệt frontend, backend, media và database.
* Không public trực tiếp ECS task ra Internet.
* Đóng gói backend bằng Docker và quản lý image bằng ECR.
* Lưu media bằng Amazon S3.
* Thu thập logs/metrics bằng CloudWatch.
* Quản lý quyền bằng AWS IAM.
* Hỗ trợ mở rộng bằng cách tăng số ECS task.
* Theo dõi chi phí và xóa tài nguyên không còn dùng.

---

### 4. Kiến trúc giải pháp

Kiến trúc BravelSport gồm ba khu vực chính:

1. Edge và Frontend.
2. VPC và Backend.
3. Dịch vụ hỗ trợ và lưu trữ dữ liệu.

![Kiến trúc AWS BravelSport](/fcj-internship-report/images/2-Proposal/bravelsport_aws_architecture.png)

#### 4.1. Edge và Frontend

Domain `bravelsport.com` được quản lý bằng Amazon Route 53. Route 53 trỏ domain tới Amazon CloudFront. AWS WAF gắn với CloudFront để lọc request. CloudFront phân phối frontend từ S3 Frontend Bucket (private, không public trực tiếp).

#### 4.2. VPC và Backend

Amazon VPC tạo ranh giới mạng cho backend. ALB và NAT Gateway nằm public subnet; ECS Fargate nằm private subnet, không có public IP. ALB chuyển API traffic tới ECS qua target group và health check. ECS chỉ nhận inbound từ security group của ALB.

Outbound từ ECS tới ECR, CloudWatch, S3, MongoDB Atlas đi qua NAT Gateway và Internet Gateway.

#### 4.3. Dịch vụ hỗ trợ và lưu trữ

* Amazon ECR — Docker image backend.
* Amazon CloudWatch — logs và metrics.
* Amazon S3 Frontend Bucket — static frontend.
* Amazon S3 Media Upload — hình ảnh/media.
* Amazon S3 Private Configuration Bucket — tệp cấu hình (nếu dùng).
* AWS IAM — quyền truy cập.
* MongoDB Atlas — dữ liệu ứng dụng.

Credential không lưu trong source code hoặc Docker image. Bucket cấu hình (nếu dùng) cần Block Public Access, mã hóa, chỉ ECS Task Role đọc được, không public URL, không dùng chung với frontend bucket, không commit lên GitHub.

Luồng DB:

`ECS Fargate → NAT Gateway → Internet Gateway → MongoDB Atlas`

Nên giới hạn Atlas theo Elastic IP của NAT Gateway, không mở `0.0.0.0/0`.

---

### 5. Các dịch vụ được sử dụng

* **Amazon Route 53** — DNS / domain.
* **Amazon CloudFront** — phân phối frontend và chuyển API tới ALB.
* **AWS WAF** — lọc request tại edge.
* **Amazon S3** — frontend, media, cấu hình.
* **Amazon VPC / Public & Private Subnet** — phân tách mạng.
* **Application Load Balancer / Target Group** — ingress API + health check.
* **NAT Gateway / Internet Gateway / Elastic IP** — outbound cho ECS.
* **Amazon ECS Fargate** — chạy backend container.
* **Amazon ECR** — registry image.
* **AWS IAM** — least privilege.
* **Amazon CloudWatch** — giám sát.
* **MongoDB Atlas** — database ngoài AWS.

---

### 6. Luồng hoạt động chính

1. Người dùng truy cập `bravelsport.com`.
2. Route 53 phân giải DNS tới CloudFront.
3. WAF kiểm tra request.
4. CloudFront lấy static frontend từ S3.
5. CloudFront chuyển request API tới ALB.
6. ALB chuyển request tới ECS Fargate.
7. ECS xử lý nghiệp vụ.
8. IAM Role cấp quyền truy cập tài nguyên AWS.
9. ECS pull image từ ECR khi khởi tạo.
10. ECS đọc/ghi media trên S3.
11. ECS gửi logs/metrics tới CloudWatch.
12. ECS kết nối MongoDB Atlas qua NAT Gateway.
13. Kết quả trả về qua ALB và CloudFront.

---

### 7. Thiết kế bảo mật

#### SG-ALB

* Inbound cần thiết tới ALB.
* Outbound tới backend port của ECS.
* Không mở cổng quản trị không cần thiết.

#### SG-ECS

* Chỉ inbound từ SG-ALB.
* Không mở backend port cho `0.0.0.0/0`.
* Outbound tới dịch vụ cần thiết.

Biện pháp khác: ECS không public IP; S3 Block Public Access; IAM least privilege; mã hóa credential; Atlas giới hạn theo NAT EIP; WAF test Count trước khi Block.

---

### 8. Lộ trình ba tháng

#### Tháng 1 – Tìm hiểu dịch vụ AWS

Tìm hiểu Route 53, CloudFront, WAF, S3, VPC, ALB, NAT, ECS Fargate, ECR, IAM, CloudWatch; phân tích yêu cầu BravelSport và sơ đồ kiến trúc ban đầu.

#### Tháng 2 – Nghiên cứu và thử nghiệm

Container hóa backend, thử ECR/ECS, frontend S3/CloudFront, thiết kế VPC/SG/ALB, kiểm tra kết nối MongoDB Atlas.

#### Tháng 3 – Triển khai và hoàn thiện

Triển khai frontend/backend/mạng; tích hợp S3, Atlas, IAM, CloudWatch; kiểm thử và hoàn thiện tài liệu workshop.

---

### 9. Chi phí và tối ưu

Nhóm chi phí chính: Route 53, CloudFront, S3, WAF, ALB, ECS Fargate, NAT Gateway, Elastic IP, ECR, CloudWatch, MongoDB Atlas.

Ước tính cụ thể dùng [AWS Pricing Calculator](https://calculator.aws/) theo Region và mức sử dụng thực tế.

Tối ưu: chọn task size phù hợp, giảm thời gian lab, xóa NAT/ALB/EIP/image cũ khi không dùng, giới hạn retention CloudWatch Logs, tối ưu cache CloudFront, clean-up sau workshop.

---

### 10. Rủi ro và giảm thiểu

* S3 bị public → Block Public Access + kiểm tra policy.
* Credential lộ → mã hóa + giới hạn Task Role; không commit GitHub.
* ECS không pull image → kiểm tra ECR, IAM, NAT.
* ALB health check fail → kiểm tra port/path/SG.
* Không nối Atlas → kiểm tra EIP, NAT route, credential.
* WAF chặn request hợp lệ → Count mode trước.
* Chi phí NAT tăng → theo dõi và clean-up.

---

### 11. Kết quả kỳ vọng

* BravelSport chạy trên AWS với domain Route 53.
* Frontend S3 + CloudFront; backend ECS Fargate private.
* API qua ALB; image trên ECR; media trên S3; data trên Atlas.
* Logs/metrics trên CloudWatch; quyền qua IAM.
* Có quy trình kiểm thử và clean-up rõ ràng.

---

### 12. Cải tiến trong tương lai

* CloudWatch Alarms, AWS Budgets, CI/CD.
* ECS Service Auto Scaling, multi-AZ.
* VPC Endpoint giảm traffic NAT.
* Secrets Manager / secret chuyên dụng.
* Backup S3 và MongoDB Atlas.

---

### 13. Kết luận

Kiến trúc tách frontend, backend, media và database giúp BravelSport triển khai rõ ràng hơn trên AWS: S3 + CloudFront cho frontend, Docker + ECS Fargate private cho backend, ALB cho API, NAT cho outbound, ECR cho image, CloudWatch cho giám sát và MongoDB Atlas cho dữ liệu. Lộ trình ba tháng hỗ trợ học dịch vụ, thử nghiệm và hoàn thiện workshop.
