---
title: "Build và push Docker image"
date: 2026-07-19
weight: 2
chapter: false
pre: " <b> 5.4.2. </b> "
---


## Mục tiêu

Build backend NestJS trên EC2 Ubuntu, kiểm tra image và push lên ECR.

## Các bước thực hiện

### 1. Clone hoặc cập nhật source

```bash
cd ~/ShopBanDoAo
git fetch --all
git checkout <COMMIT_OR_BRANCH>
git rev-parse --short HEAD
```


### 2. Build Docker image

```bash
cd backend-nestjs
docker build -t bravelsport-backend:<IMAGE_TAG> .
```

Không copy `.env` hoặc secret vào image. Kiểm tra `.dockerignore`.

### 3. Kiểm tra image local

```bash
docker image inspect bravelsport-backend:<IMAGE_TAG>
docker run --rm -p 3000:3000 --env-file <SAFE_ENV_FILE> \
  bravelsport-backend:<IMAGE_TAG>
curl -i http://localhost:3000/
```

### 4. Đăng nhập ECR

```bash
aws ecr get-login-password --region <AWSx1_REGION> \
| docker login --username AWS --password-stdin \
  <ACCOUNT_ID>.dkr.ecr.<AWS_REGION>.amazonaws.com
```

Không ghi Account ID thật vào tài liệu hoặc ảnh.

### 5. Tag và push

```bash
docker tag bravelsport-backend:<IMAGE_TAG> \
  <ECR_REPOSITORY_URI>:<IMAGE_TAG>

docker push <ECR_REPOSITORY_URI>:<IMAGE_TAG>
```

<!-- CHÈN ẢNH TẠI ĐÂY:
Chụp kết quả docker build và docker push hoàn tất. Che Account ID, repository URI đầy đủ và token đăng nhập.
Thông tin cần che: Account ID, ARN, địa chỉ IP công khai, token, cookie, secret và dữ liệu người dùng nếu xuất hiện.
-->

![Build và push backend image](/images/5-Workshop/5.4-Backend-deployment/build-and-push-image.png)

## Kiểm tra kết quả

Amazon ECR Repository `web-ban-ao-backend` đã lưu Docker image của backend thành công.

Thông tin image hiện tại:

| Thông số | Giá trị |
| --- | --- |
| Repository | `web-ban-ao-backend` |
| Image tag | `latest` |
| Image size | `83.38 MB` |
| Image digest | `sha256:24ba0e01ba...` |
| Trạng thái | Image đã được push lên ECR thành công |
| CPU architecture | Cần kiểm tra bằng Docker CLI |
| Backend port | `3000` |
| Health check path | `/` |


![Docker image trong Amazon ECR](/images/5-Workshop/5.4-Backend-deployment/ecr-backend-image.png)
## Lỗi thường gặp

| Lỗi | Cách xử lý |
|---|---|
| `no basic auth credentials` | Chạy lại ECR login đúng Region |
| Push denied | Kiểm tra EC2 IAM Role và repository policy |
| Image quá lớn | Dùng multi-stage build và loại dev dependency |
| Container chỉ bind localhost | Đảm bảo ứng dụng lắng nghe interface phù hợp trong container |

## Tóm tắt

Backend image đã được build, kiểm tra và lưu trong ECR.


- Phần trước: [5.4.1. Tạo ECR](../5.4.1-Create-ecr/)
- Phần tiếp theo: [5.4.3. Tạo IAM Role](../5.4.3-Create-iam-roles/)
