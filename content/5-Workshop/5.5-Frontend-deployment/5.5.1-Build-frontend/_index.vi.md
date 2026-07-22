---
title: "Build frontend React/Vite"
date: 2026-07-19
weight: 1
chapter: false
pre: " <b> 5.5.1. </b> "
---



## Mục tiêu

Cấu hình production URL và build static assets trên EC2 Build Machine.

## Các bước thực hiện

1. Trên máy build (EC2 hoặc máy phát triển):

```bash
cd ~/ShopBanDoAo/frontend
npm ci
```

2. Cấu hình API Base URL để frontend gọi backend thông qua cùng domain CloudFront:

```text
VITE_API_BASE_URL=https://bravelsport.com/api
```

Nếu triển khai với tên miền khác, thay `bravelsport.com` bằng tên miền tương ứng.

3. Cấu hình Socket.IO Client

Socket.IO Client kết nối đến cùng origin của website và sử dụng namespace:

```text
/chat
```

Transport path:

```text
/socket.io
```

Không cấu hình path thành `/chat` vì `/chat` chỉ là namespace của Socket.IO.

4. Build frontend:

```bash
npm run build
```

Sau khi build thành công, kiểm tra thư mục đầu ra:

```bash
ls -la dist
```

Kết quả sẽ sinh thư mục `dist/` chứa các file HTML, CSS và JavaScript tối ưu cho môi trường production.


5. Kiểm tra bundle không chứa thông tin nhạy cảm:

```bash
rg -n "MONGO_URI|JWT_SECRET|API_SECRET|PRIVATE_KEY" dist || true
```

Nếu lệnh không trả về kết quả, bundle frontend không chứa các secret và sẵn sàng triển khai lên Amazon S3.

<!-- CHÈN ẢNH TẠI ĐÂY:
Chụp npm run build thành công và thư mục dist. Che domain staging hoặc token nếu có.
Thông tin cần che: Account ID, ARN, địa chỉ IP công khai, token, cookie, secret và dữ liệu người dùng nếu xuất hiện.
-->

![Build React Vite frontend](/images/5-Workshop/5.5-Frontend-deployment/build-frontend.jpg)

## Kiểm tra kết quả

- `dist/` tồn tại.
- API không trỏ `localhost:3000`.
- Socket.IO path/namespace được cấu hình đúng.
- Không có secret trong bundle.

## Lỗi thường gặp

| Lỗi | Cách xử lý |
|---|---|
| Build dùng env cũ | Xóa dist và build lại sau khi sửa env |
| URL thành `/api/api` | Kiểm tra base URL và endpoint trong Axios service |
| Socket.IO dùng `/chat` làm path | Dùng `/chat` làm namespace, transport path là `/socket.io` |

## Tóm tắt

Frontend production bundle đã sẵn sàng để upload lên S3.


- Phần trước: [5.5. Triển khai frontend](../)
- Phần tiếp theo: [5.5.2. Tạo S3 Frontend Bucket](../5.5.2-Create-s3-frontend/)
