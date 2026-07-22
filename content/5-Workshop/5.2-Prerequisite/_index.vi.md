---
title: "Các bước chuẩn bị"
date: 2026-07-19
weight: 2
chapter: false
pre: " <b> 5.2. </b> "
---



## Mục tiêu

Chuẩn bị tài khoản, IAM identity, EC2 Ubuntu Build Machine, công cụ, source code, MongoDB Atlas, DNS, certificate và bảng thông số trước khi tạo tài nguyên AWS.

## Các bước thực hiện

### 1. Chuẩn bị tài khoản và IAM Identity

- Không sử dụng root user cho thao tác hằng ngày.
- Tạo hoặc sử dụng IAM Identity có quyền phù hợp với workshop.
- Bật MFA nếu có thể.
- Gắn IAM Role qua Instance Profile cho EC2 thay vì lưu access key lâu dài.
- Không cấp `AdministratorAccess` nếu nhóm có thể tạo policy giới hạn hơn.

Kiểm tra identity từ EC2:

```bash
aws sts get-caller-identity
```


### 2. Chọn AWS Region

Chọn Region cho VPC, EC2, ECR, ECS, ALB, S3 Media và CloudWatch: `[TO BE CONFIRMED]`.

Certificate dùng cho CloudFront phải được tạo hoặc import trong `us-east-1`. Không tự điền Region khác vào tài liệu khi nhóm chưa xác nhận.

### 3. Tạo EC2 Ubuntu Build Machine

Tạo EC2 Ubuntu trong môi trường Development & Deployment.

Yêu cầu:

- Instance type: `[TO BE CONFIRMED]`.
- Storage: `[TO BE CONFIRMED]`.
- IAM instance profile: quyền tối thiểu để push ECR, upload S3 và chạy lệnh triển khai cần thiết.
- Không mở cổng `3000`.
- Nếu dùng SSH, chỉ mở cổng `22` từ IP quản trị `[TO BE CONFIRMED]`.
- Ưu tiên Systems Manager Session Manager nếu tài khoản và AMI đã cấu hình phù hợp.


![EC2 Build Machine đang chạy](/images/5-Workshop/5.2-Prerequisite/ec2-running.jpg)

### 4. Cài đặt công cụ

Trên Ubuntu:

```bash
sudo apt update
sudo apt install -y git unzip ca-certificates curl

aws --version
docker --version
node --version
npm --version
git --version
```

Cài phiên bản Node.js phù hợp với `package.json` và xác nhận Docker daemon đang hoạt động. 

![Verify local tools](/images/5-Workshop/5.2-Prerequisite/verify-local-tools.jpg)

### 5. Clone mã nguồn

```bash
git clone https://github.com/huu-luan-2004/ShopBanDoAo.git
cd ShopBanDoAo
```

Ghi lại commit dùng cho workshop:

```bash
git rev-parse --short HEAD
```


### 6. Build thử frontend

```bash
cd Fronted
npm ci
npm run build
ls -la dist
```

Không đưa secret vào biến có tiền tố `VITE_` vì các biến này có thể được đóng gói vào static bundle.

<!-- CHÈN ẢNH TẠI ĐÂY:
Chụp terminal kết thúc npm run build và danh sách file trong dist. Che URL nội bộ hoặc token nếu xuất hiện.
Thông tin cần che: Account ID, ARN, địa chỉ IP công khai, token, cookie, secret và dữ liệu người dùng nếu xuất hiện.
-->

<!-- Ảnh chưa có trong static: /images/5-Workshop/5.2-Prerequisite/frontend-build-success.png -->

### 7. Build thử Docker image backend

Từ thư mục chứa Dockerfile phù hợp:

```bash
cd ../backend-nestjs
docker build -t bravelsport-backend:local .
docker image ls bravelsport-backend
```

Chạy thử container với biến môi trường giả hoặc môi trường test an toàn:

```bash
docker run --rm -p 3000:3000 \
  --env-file <SAFE_ENV_FILE> \
  bravelsport-backend:local
```

Kiểm tra:

```bash
curl -i http://localhost:3000/
```

Health endpoint ban đầu là `/`. Nếu ứng dụng trả mã khác `200`, phải xác nhận route thật trước khi tạo Target Group.


![Docker image build thành công](/images/5-Workshop/5.2-Prerequisite/backend-docker-build-success.jpg)

### 8. Chuẩn bị MongoDB Atlas

- Tạo cluster.
- Tạo database user riêng cho ứng dụng với quyền tối thiểu.
- Chưa mở `0.0.0.0/0` cho cấu hình cuối.
- Sau khi có NAT Gateway, thêm Elastic IP của NAT vào IP Access List.
- Không đưa connection string vào ảnh hoặc tài liệu.



![MongoDB Atlas Network Access](/images/5-Workshop/5.2-Prerequisite/mongodb-network-access.jpg)

### 9. Chuẩn bị Route 53 Hosted Zone

Xác nhận Hosted Zone cho domain `bravelsport.com`. Không thay đổi record đang phục vụ production khi chưa có kế hoạch chuyển đổi.


![Route 53 Hosted Zone](/images/5-Workshop/5.2-Prerequisite/route53-hosted-zone.jpg)

### 10. Chuẩn bị ACM certificate

Trong Region `us-east-1`, request certificate cho hostname dùng với CloudFront. Hoàn tất DNS validation và chờ trạng thái `Issued`.

![ACM certificate Issued](/images/5-Workshop/5.2-Prerequisite/acm-certificate-issued.jpg)


## Kiểm tra kết quả

- `aws sts get-caller-identity` chạy thành công.
- Docker daemon hoạt động.
- `npm run build` tạo thư mục `dist/`.
- Docker image backend chạy và lắng nghe cổng `3000`.
- Health path `/` đã được kiểm tra hoặc được đánh dấu cần xác nhận.
- Hosted Zone và ACM certificate sẵn sàng.
- Không có secret trong terminal, ảnh hoặc Git history.

## Lỗi thường gặp

| Lỗi | Nguyên nhân | Cách xử lý |
|---|---|---|
| AWS CLI AccessDenied | IAM identity thiếu quyền | Bổ sung quyền tối thiểu hoặc kiểm tra instance profile |
| Docker permission denied | User chưa thuộc nhóm docker | Dùng `sudo` tạm thời hoặc cấu hình nhóm docker đúng quy trình |
| Frontend build lỗi | Node/npm không phù hợp hoặc thiếu env | Đối chiếu package-lock và `.env.example` |
| Container không mở cổng 3000 | Dockerfile hoặc biến `PORT` sai | Kiểm tra log và port mapping |
| ACM không Issued | DNS validation chưa hoàn tất | Kiểm tra CNAME validation trong Route 53 |

## Tóm tắt

Môi trường build, source code, domain, certificate và các giá trị triển khai đã được chuẩn bị. EC2 chỉ phục vụ build/deployment, không tham gia runtime request.



- Phần trước: [5.1. Tổng quan workshop](../5.1-Workshop-overview/)
- Phần tiếp theo: [5.3. Xây dựng hạ tầng mạng](../5.3-Network-infrastructure/)
