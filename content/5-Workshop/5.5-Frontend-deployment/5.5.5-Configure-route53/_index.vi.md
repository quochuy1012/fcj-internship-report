---
title: "Cấu hình Route 53"
date: 2026-07-19
weight: 5
chapter: false
pre: " <b> 5.5.5. </b> "
---



## Mục tiêu

Trỏ domain workshop đến CloudFront và phục vụ HTTPS bằng ACM certificate tại `us-east-1`.

## Các bước thực hiện

1. Trong Amazon CloudFront, thêm **Alternate Domain Name (CNAME)**:

```text
bravelsport.com
```

2. Chọn ACM Certificate có trạng thái:

```text
Issued
```

Certificate phải được tạo tại Region:

```text
us-east-1 (N. Virginia)
```

vì CloudFront chỉ hỗ trợ ACM Certificate tại Region này.

3. Chọn **Save changes** và chờ CloudFront Distribution chuyển sang trạng thái:

```text
Deployed
```

4. Truy cập:

```text
Amazon Route 53 → Hosted Zones → bravelsport.com
```

5. Tạo bản ghi:

| Record | Type | Alias | Giá trị |
|---|---|---|---|
| `bravelsport.com` | A | Yes | CloudFront Distribution |

Nếu website hỗ trợ IPv6, tạo thêm:

| Record | Type | Alias | Giá trị |
|---|---|---|---|
| `bravelsport.com` | AAAA | Yes | CloudFront Distribution |

6. Không trỏ trực tiếp DNS của người dùng đến:

- Amazon S3 Bucket
- Application Load Balancer
- Amazon ECS

Toàn bộ request phải đi qua CloudFront để sử dụng:

- Origin Access Control (OAC)
- AWS WAF
- CloudFront Cache
- HTTPS

7. Sau khi DNS cập nhật, kiểm tra:

- `https://bravelsport.com`
- Frontend được tải từ CloudFront.
- REST API hoạt động qua `/api/*`.
- Socket.IO hoạt động qua `/socket.io/*`.
- HTTPS hoạt động với ACM Certificate.

<!-- CHÈN ẢNH TẠI ĐÂY:
Chụp Hosted Zone của Route 53.

Ảnh cần hiển thị:

- Hosted Zone: bravelsport.com
- Record A (Alias) trỏ đến CloudFront
- NS Record
- SOA Record
- CNAME xác thực ACM (nếu có)

Che Distribution Domain Name, Account ID và các giá trị không cần thiết.
-->


-->

![Configure Route 53 alias](/images/5-Workshop/5.5-Frontend-deployment/configure-route53.png)

## Kiểm tra kết quả

```bash
nslookup <WORKSHOP_DOMAIN>
curl -I https://<WORKSHOP_DOMAIN>/
```

- DNS phân giải đến CloudFront.
- HTTPS certificate hợp lệ.
- Không lộ ALB DNS trong URL người dùng.

## Lỗi thường gặp

| Lỗi | Cách xử lý |
|---|---|
| Certificate không chọn được | Tạo certificate tại `us-east-1` và xác thực domain |
| DNS vẫn trỏ hệ thống cũ | Kiểm tra record/TTL và kế hoạch cutover |
| 502 từ CloudFront | Kiểm tra ALB origin, listener, SG và target health |

## Tóm tắt

Domain đã trỏ đến CloudFront, hoàn tất đường vào frontend và backend.

## Danh sách ảnh

| STT | Tên file | Nội dung ảnh | Vị trí chèn |
|---:|---|---|---|
| 1 | `configure-route53.png` | Route 53 Alias đến CloudFront | Sau bước tạo record |

## Điều hướng

- Phần trước: [5.5.4. Cấu hình WAF](../5.5.4-Configure-waf/)
- Phần tiếp theo: [5.6. Database và media](../../5.6-Data-and-media/)
