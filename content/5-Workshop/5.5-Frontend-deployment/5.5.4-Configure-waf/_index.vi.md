---
title: "Cấu hình AWS WAF"
date: 2026-07-19
weight: 4
chapter: false
pre: " <b> 5.5.4. </b> "
---



## Mục tiêu

Liên kết AWS WAF Web ACL với Amazon CloudFront để bảo vệ frontend và backend khỏi các request độc hại trước khi được chuyển đến Application Load Balancer hoặc Amazon S3.

## Các bước thực hiện

1. Truy cập **AWS WAF & Shield**.

2. Chọn scope:

```text
CloudFront (Global)
```

3. Tạo Web ACL:

```text
CreatedByCloudFront-7306abc9
```

Sau khi tạo, Associate Web ACL với CloudFront Distribution.

4. Thêm các AWS Managed Rules.

Trong workshop sử dụng các Managed Rule phổ biến:

- AWSManagedRulesCommonRuleSet
- AWSManagedRulesKnownBadInputsRuleSet
- AWSManagedRulesAmazonIpReputationList

5. Trong giai đoạn kiểm thử, có thể đặt các rule có nguy cơ false positive ở chế độ:

```text
Count
```

Sau khi xác nhận ứng dụng hoạt động ổn định, chuyển sang:

```text
Block
```

6. Bật:

- Sampled Requests
- CloudWatch Metrics

để theo dõi lưu lượng và kiểm tra các request bị WAF xử lý.

7. Kiểm thử các chức năng của hệ thống:

- Đăng nhập.
- Đăng ký.
- REST API.
- Upload hình ảnh.
- Socket.IO.
- React Router.

Đảm bảo các request hợp lệ không bị WAF chặn trước khi chuyển toàn bộ rule sang chế độ **Block**.

Không áp dụng các rule production nếu chưa hoàn thành quá trình kiểm thử.

<!-- CHÈN ẢNH TẠI ĐÂY:
Chụp Web ACL sau khi tạo.
Ảnh cần hiển thị:
- Web ACL Name: CreatedByCloudFront-7306abc9
- Scope: CloudFront
- Associated CloudFront Distribution
- 3 AWS Managed Rules

Che Account ID, ARN và Distribution ID.
-->

![AWS WAF Web ACL](/images/5-Workshop/5.5-Frontend-deployment/configure-waf.png)

## Kiểm tra kết quả

- Web ACL được Associate với CloudFront Distribution.
- Scope là **CloudFront (Global)**.
- Web ACL chứa **3 AWS Managed Rules**.
- Các request hợp lệ vẫn truy cập website bình thường.
- Sampled Requests hoạt động.
- CloudWatch Metrics được ghi nhận.
- Các rule chỉ chuyển sang **Block** sau khi hoàn tất kiểm thử.
## Lỗi thường gặp

| Lỗi | Cách xử lý |
|---|---|
| Login/upload bị chặn | Xem sampled request và điều chỉnh rule/exclusion |
| Không thấy distribution | Kiểm tra scope CloudFront Global |
| Chi phí tăng | Giới hạn số rule và theo dõi request volume |

## Tóm tắt

WAF đã được triển khai theo hướng quan sát trước, chặn sau.



- Phần trước: [5.5.3. Tạo CloudFront](../5.5.3-Create-cloudfront/)
- Phần tiếp theo: [5.5.5. Cấu hình Route 53](../5.5.5-Configure-route53/)
