---
title: "Blog 3"
date: 2024-01-01
weight: 3
chapter: false
pre: " <b> 3.3. </b> "
---

# Hiện đại hóa SignalR với AWS AppSync Event API

Tôi tìm hiểu chủ đề **Modernizing SignalR with AWS AppSync Event API** — cách hiện đại hóa realtime của ứng dụng .NET/React bằng cách chuyển từ SignalR truyền thống sang **AWS AppSync Event API**.

Với app realtime (thông báo đơn hàng, live dashboard, trạng thái giao hàng, chat, pub/sub), SignalR thường giữ WebSocket giữa server và client. Khi scale, team dễ gặp việc phải quản lý nhiều connection, giữ server luôn chạy, sticky session hoặc Redis backplane.

## Kiến trúc mới

![Kiến trúc AWS AppSync Event API](/images/3-BlogsPosted/blog3.png)

> Nguồn bài: [AWS Study Group — Facebook](https://www.facebook.com/groups/awsstudygroupfcj/permalink/2206455833452710/)

Bốn luồng chính:

1. **React Client** gọi **.NET API Server** khi có hành động cần realtime.
2. **.NET API Server** lấy API key/endpoint từ **AWS Secrets Manager** thay vì hard-code.
3. Backend publish event lên **AWS AppSync Event API** qua HTTP.
4. **AppSync** đẩy event realtime tới subscribers qua WebSocket (pub/sub).

## Điểm khác so với SignalR truyền thống

- **SignalR:** server vừa xử lý nghiệp vụ, vừa quản lý WebSocket connection.
- **AppSync Event API:** server tập trung business logic và publish event; AWS lo phần kết nối, phân phối và scale.

Phù hợp với thông báo realtime, live dashboard, cập nhật đơn hàng, collaborative app, IoT fan-out…

Secrets Manager giúp tách credential khỏi source code. Triển khai thực tế có thể thêm IAM, CloudWatch, CloudTrail và tách môi trường Dev/Staging/Prod.

## Kết luận

Hiện đại hóa SignalR không phải bỏ realtime, mà chuyển phần hạ tầng WebSocket sang dịch vụ managed/serverless của AWS. Backend gọn hơn, frontend vẫn nhận cập nhật tức thì, còn AWS lo phần kết nối phía sau.
