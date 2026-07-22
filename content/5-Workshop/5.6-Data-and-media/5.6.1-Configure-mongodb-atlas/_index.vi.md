---
title: "Cấu hình MongoDB Atlas"
date: 2026-07-19
weight: 1
chapter: false
pre: " <b> 5.6.1. </b> "
---


## Mục tiêu

Cho phép Amazon ECS Fargate kết nối đến MongoDB Atlas thông qua NAT Gateway, đồng thời giới hạn quyền truy cập bằng IP Access List và quản lý chuỗi kết nối bằng AWS Secrets Manager.

## Các bước thực hiện

1. Đăng nhập MongoDB Atlas và xác nhận Cluster đang ở trạng thái **Available**.

2. Tạo Database User riêng cho ứng dụng BravelSport với quyền tối thiểu (`Read and Write to Any Database` hoặc quyền phù hợp với ứng dụng).

3. Truy cập:

```text
Network Access → IP Access List
```

4. Thêm Elastic IP của NAT Gateway theo định dạng:

```text
<NAT_GATEWAY_ELASTIC_IP>/32
```

Chỉ cho phép địa chỉ IP Public của NAT Gateway truy cập MongoDB Atlas.

5. Sau khi hoàn tất quá trình kiểm thử, xóa rule:

```text
0.0.0.0/0
```

để tránh cho phép truy cập từ toàn bộ Internet.

6. Chỉ giữ các địa chỉ IP quản trị viên khi thật sự cần thiết.

7. Tạo MongoDB Connection String.

Ví dụ:

```text
mongodb+srv://<username>:<password>@cluster.mongodb.net/<database>
```

Không đưa Connection String hoặc mật khẩu vào tài liệu workshop.

8. Lưu biến:

```text
MONGO_URI
```

trong:

```text
AWS Secrets Manager
```

Sau đó tham chiếu secret này trong ECS Task Definition.

9. Đăng ký Task Definition Revision mới và cập nhật ECS Service để backend sử dụng Connection String mới.

<!-- CHÈN ẢNH TẠI ĐÂY:
Chụp MongoDB Atlas → Network Access → IP Access List.

Ảnh cần hiển thị:
- NAT Gateway Elastic IP (/32)
- Trạng thái Active

Che:
- Public IP đầy đủ
- Project ID
- Username
- Connection String
-->

![MongoDB Atlas IP Access List](/images/5-Workshop/5.6-Data-and-media/configure-mongodb-atlas.jpg)




## Kiểm tra kết quả

- MongoDB Atlas chỉ cho phép Elastic IP của NAT Gateway truy cập.
- Rule `0.0.0.0/0` đã được xóa sau khi hoàn tất kiểm thử.
- ECS Service cập nhật thành công với Task Definition Revision mới.
- Backend kết nối thành công tới MongoDB Atlas.
- API đọc và ghi dữ liệu bình thường.
- CloudWatch Logs không chứa Connection String hoặc mật khẩu.

## Lỗi thường gặp

| Lỗi | Cách xử lý |
|---|---|
| Server selection timeout | Kiểm tra Elastic IP của NAT Gateway, Route Table, NAT Gateway và Connection String |
| Authentication failed | Kiểm tra Database User, mật khẩu, `authSource` và URL Encoding |
| ECS không đọc được secret | Kiểm tra `ecsTaskExecutionRole` và quyền `secretsmanager:GetSecretValue` |
| Backend không kết nối được Atlas | Kiểm tra IP Access List và Security Group outbound |
| Connection String xuất hiện trong log | Loại bỏ log nhạy cảm, cập nhật Secret và triển khai lại ECS Service |

## Lỗi thường gặp

| Lỗi | Cách xử lý |
|---|---|
| Server selection timeout | Kiểm tra EIP, NAT, DNS và connection string |
| Authentication failed | Kiểm tra database user, authSource và URL encoding |
| Secret không inject | Kiểm tra execution role và secret ARN/name |
| URI xuất hiện trong log | Sửa logging, rotate secret và xử lý log theo chính sách |

## Tóm tắt

ECS đã kết nối MongoDB Atlas qua địa chỉ outbound cố định của NAT Gateway.



## Điều hướng

- Phần trước: [5.6. Database và media](../)
- Phần tiếp theo: [5.6.2. Tạo S3 Media](../5.6.2-Create-media-bucket/)
