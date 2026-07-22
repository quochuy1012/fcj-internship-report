---
title: "Tạo S3 Frontend Bucket"
date: 2026-07-19
weight: 2
chapter: false
pre: " <b> 5.5.2. </b> "
---




## Mục tiêu

Lưu trữ frontend tĩnh trên Amazon S3 bằng bucket riêng tư (private bucket) và phân phối nội dung thông qua Amazon CloudFront với Origin Access Control (OAC). Không sử dụng S3 Static Website Hosting.

## Các bước thực hiện

1. Tạo bucket:

| Thông số | Giá trị |
|---|---|
| Bucket name | `myapp-frontend-sport` |
| AWS Region | `ap-southeast-1 (Singapore)` |

2. Bật toàn bộ **Block Public Access**.

3. Bật **Default Encryption (SSE-S3)** để mã hóa dữ liệu mặc định.

4. Bật **Bucket Versioning** nếu cần hỗ trợ khôi phục phiên bản trước của website.

5. Không bật **Static Website Hosting** vì frontend sẽ được truy cập thông qua CloudFront sử dụng Origin Access Control (OAC).

<!-- CHÈN ẢNH TẠI ĐÂY:
Chụp tab Permissions của bucket.
Ảnh cần hiển thị:
- Bucket name: myapp-frontend-sport
- Block Public Access: On

Che Account ID nếu xuất hiện.
-->

![Tạo S3 Frontend Bucket - bước 1](/images/5-Workshop/5.5-Frontend-deployment/create-s3-frontend1.png)

6. Upload toàn bộ nội dung thư mục `dist/` lên bucket:

```bash
aws s3 sync dist/ s3://myapp-frontend-sport/ --delete
```

Lệnh trên sẽ:

- Đồng bộ toàn bộ nội dung thư mục `dist`.
- Tự động cập nhật file thay đổi.
- Xóa các file không còn tồn tại trong thư mục `dist`.

<!-- CHÈN ẢNH TẠI ĐÂY:
Chụp tab Objects của bucket.
Ảnh cần hiển thị:
- index.html
- assets/
- api/
- Các file frontend đã upload

Che Account ID nếu xuất hiện.
-->

![Tạo S3 Frontend Bucket - bước 2](/images/5-Workshop/5.5-Frontend-deployment/create-s3-frontend2.png)

7. Cấu hình Cache-Control cho các đối tượng trong bucket:

- `index.html`

```text
Cache-Control: no-cache, no-store, must-revalidate
```

- Các file CSS, JavaScript và hình ảnh có hash trong thư mục `assets/`

```text
Cache-Control: public, max-age=31536000, immutable
```

Cấu hình này giúp:

- `index.html` luôn được CloudFront kiểm tra phiên bản mới.
- Các file tĩnh có hash được cache lâu để tăng hiệu năng và giảm chi phí truy cập.

<!-- CHÈN ẢNH TẠI ĐÂY:
Chụp Block Public Access và danh sách objects. Che bucket ARN/owner và object có dữ liệu không liên quan.
Thông tin cần che: Account ID, ARN, địa chỉ IP công khai, token, cookie, secret và dữ liệu người dùng nếu xuất hiện.
-->



## Kiểm tra kết quả

- Bucket private.
- `index.html` và assets tồn tại.
- Direct S3 object URL bị từ chối nếu không có quyền.
- Không có `.env` hoặc source map nhạy cảm ngoài kế hoạch.

## Lỗi thường gặp

| Lỗi | Cách xử lý |
|---|---|
| Website 403 qua CloudFront | Cấu hình OAC và bucket policy ở bước sau |
| Upload sai cấp thư mục | Sync nội dung bên trong `dist/`, không phải thư mục `dist` lồng nhau |
| Bucket public | Bật lại Block Public Access và xóa public ACL/policy |

## Tóm tắt

Static frontend đã được lưu an toàn trong S3.


## Điều hướng

- Phần trước: [5.5.1. Build frontend](../5.5.1-Build-frontend/)
- Phần tiếp theo: [5.5.3. Tạo CloudFront](../5.5.3-Create-cloudfront/)
