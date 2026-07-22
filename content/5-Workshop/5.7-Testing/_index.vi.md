---
title: "Kiểm thử end-to-end"
date: 2026-07-19
weight: 7
chapter: false
pre: " <b> 5.7. </b> "
---


## Mục tiêu

Xác minh toàn bộ luồng hoạt động của hệ thống BravelSport từ người dùng đến CloudFront, Application Load Balancer, ECS Fargate, MongoDB Atlas, Amazon S3 và Amazon CloudWatch.

Việc kiểm thử phải dựa trên response thực tế, log hệ thống và ảnh minh chứng. Không đánh dấu **PASS** khi chưa có bằng chứng phù hợp.

## Các bước thực hiện

### 1. Ghi nhận phiên bản kiểm thử

Ghi lại các thông tin của phiên bản đang được kiểm thử để có thể đối chiếu khi xảy ra lỗi hoặc cần rollback.

| Thông tin | Giá trị |
|---|---|
| Thời điểm kiểm thử | `22/07/2026` |
| Domain | `bravelsport.com` |
| AWS Region | `ap-southeast-1` |
| ECS Cluster | `web-ban-ao-cluster` |
| ECS Service | `web-ban-ao-backend-service` |
| Task Definition | `web-ban-ao-backend-task:18` |
| Container image | `web-ban-ao-backend:latest` |
| Container port | `3000` |
| Frontend bucket | `myapp-frontend-sport` |
| Media bucket | `bravel-uploads` |

Đối với commit hash và image digest, có thể kiểm tra trực tiếp trong GitHub và Amazon ECR tại thời điểm triển khai.

Không cần đưa đầy đủ Account ID, ARN hoặc image digest vào tài liệu công khai.

### 2. Kiểm tra DNS và HTTPS

Thực hiện kiểm tra DNS:

```bash
nslookup bravelsport.com
```


Kiểm tra HTTPS:

```bash
curl -I https://bravelsport.com/
```

Kết quả mong đợi:

- Domain `bravelsport.com` được phân giải thành CloudFront Distribution.
- Kết nối sử dụng HTTPS.
- Certificate hợp lệ.
- Website trả về HTTP status thành công.
- Người dùng không truy cập trực tiếp vào S3, ALB hoặc ECS.




### 3. Kiểm tra frontend và CloudFront

Truy cập:

```text
https://bravelsport.com
```

Thực hiện các bước:

1. Kiểm tra trang chủ tải thành công.
2. Kiểm tra các file CSS, JavaScript và hình ảnh tải bình thường.
3. Mở một route React Router như:

```text
https://bravelsport.com/admin
```

4. Tải lại trang bằng trình duyệt.
5. Xác nhận không xuất hiện lỗi `403` hoặc `404`.
6. Kiểm tra direct S3 access bị từ chối.

Kết quả mong đợi:

- Frontend React/Vite được tải từ CloudFront.
- Static assets được lấy từ S3 Private Bucket thông qua OAC.
- React Router fallback hoạt động.
- S3 Bucket vẫn bật Block Public Access.

### 4. Kiểm tra AWS WAF

Trong AWS Management Console, truy cập:

```text
AWS WAF & Shield → Protection packs (web ACLs)
```

Chọn Web ACL đang liên kết với CloudFront Distribution.

Thực hiện:

- Kiểm tra Web ACL có đúng scope `CloudFront`.
- Kiểm tra CloudFront Distribution đã được associate.
- Mở phần Sampled Requests.
- Kiểm tra các request hợp lệ không bị chặn.
- Theo dõi CloudWatch Metrics của Web ACL.
- Chỉ chuyển rule từ `Count` sang `Block` sau khi hoàn tất kiểm thử.

Không thực hiện các hành vi tấn công hệ thống thật. Chỉ dùng request kiểm thử an toàn.

### 5. Kiểm tra Authentication và JWT

Thực hiện kiểm thử bằng tài khoản thử nghiệm:

1. Đăng ký tài khoản mới.
2. Đăng nhập bằng thông tin hợp lệ.
3. Gọi protected API sau khi đăng nhập.
4. Kiểm tra request có JWT hợp lệ được backend chấp nhận.
5. Kiểm tra JWT không hợp lệ hoặc hết hạn bị từ chối.
6. Đăng xuất và kiểm tra protected route không còn truy cập được.

Kết quả mong đợi:

- Đăng nhập thành công.
- Protected API trả dữ liệu khi có token hợp lệ.
- Token sai hoặc hết hạn trả mã lỗi phù hợp như `401 Unauthorized`.
- Không hiển thị JWT đầy đủ trong ảnh hoặc log.

### 6. Kiểm tra REST API `/api/*`

Mở Chrome DevTools:

```text
F12 → Network → Fetch/XHR
```

Thực hiện các thao tác trong giao diện như:

- Đăng nhập.
- Mở trang quản trị.
- Xem danh sách người dùng.
- Xem đơn hàng.
- Xem thống kê.
- Xem dữ liệu sân hoặc voucher.

Kiểm tra các request có path:

```text
/api/*
```

Kết quả mong đợi:

- Request đi qua domain `bravelsport.com`.
- Request không gọi trực tiếp ALB DNS.
- HTTP status là `200` đối với request hợp lệ.
- Request giữ nguyên path `/api/*`.
- CloudFront chuyển tiếp request đến ALB.
- ALB chuyển request đến ECS Fargate.

<!-- CHÈN ẢNH TẠI ĐÂY:
Chụp website BravelSport cùng Chrome DevTools Network.

Ảnh cần hiển thị:
- Domain bravelsport.com
- Trang quản trị hoạt động
- Các request Fetch/XHR
- HTTP Status 200
- Các request như profile, active hoặc stats

Che:
- JWT
- Cookie
- Authorization Header
- Request payload nhạy cảm
- Dữ liệu người dùng nhạy cảm
-->

![Kiểm thử end-to-end trên trình duyệt](/images/5-Workshop/5.7-Testing/end-to-end-browser-test.jpg)

```text
https://bravelsport.com/admin
```

Các request API trong DevTools Network trả về mã trạng thái `200`, chứng minh frontend đã giao tiếp thành công với backend thông qua CloudFront và Application Load Balancer.

### 7. Kiểm tra Socket.IO

Socket.IO của BravelSport sử dụng:

```text
Transport path: /socket.io/*
Namespace: /chat
```

Cần phân biệt rõ:

- `/socket.io/*` là đường dẫn transport.
- `/chat` là namespace của ứng dụng.
- Không tạo CloudFront behavior `/chat`.

Thực hiện kiểm thử:

1. Đăng nhập bằng hai tài khoản thử nghiệm.
2. Mở chức năng chat trên hai trình duyệt hoặc hai cửa sổ ẩn danh.
3. Mở Chrome DevTools.
4. Chọn tab:

```text
Network → Socket
```

5. Kiểm tra handshake Socket.IO.
6. Gửi tin nhắn từ phiên thứ nhất.
7. Xác nhận phiên thứ hai nhận được tin nhắn mà không cần reload.
8. Kiểm tra kết nối sử dụng domain HTTPS/WSS.

Kết quả mong đợi:

- Handshake thành công.
- Request sử dụng `/socket.io/*`.
- Namespace `/chat` kết nối thành công.
- JWT handshake được backend xử lý.
- Tin nhắn realtime được gửi và nhận.
- CloudFront không cache traffic Socket.IO.

### 8. Kiểm tra MongoDB Atlas

Thực hiện một thao tác tạo hoặc cập nhật dữ liệu thử nghiệm:

- Tạo người dùng.
- Tạo đơn hàng.
- Tạo voucher.
- Tạo sân.
- Cập nhật hồ sơ.
- Thực hiện thao tác khác phù hợp với hệ thống.

Sau đó mở MongoDB Atlas và xác nhận dữ liệu đã được lưu.

Kết quả mong đợi:

- ECS Fargate kết nối MongoDB Atlas thành công.
- Request đọc và ghi dữ liệu hoạt động.
- IP Access List cho phép NAT Gateway Elastic IP.
- Không cần mở `0.0.0.0/0`.
- Connection string không xuất hiện trong ảnh hoặc log.

### 9. Kiểm tra media trên Amazon S3

Thực hiện upload một file hợp lệ từ giao diện ứng dụng.

Kiểm tra:

1. Request upload trả về thành công.
2. Object mới xuất hiện trong bucket:

```text
bravel-uploads
```

3. MongoDB lưu Object Key hoặc URL tương ứng.
4. Hình ảnh hoặc file media hiển thị trên website.
5. Refresh trình duyệt và kiểm tra media vẫn còn.
6. Thay thế ECS Task và kiểm tra media không bị mất.

Kết quả mong đợi:

- File không được lưu trong local filesystem của Fargate.
- Media vẫn tồn tại sau deployment.
- ECS Task Role có quyền S3 phù hợp.
- Bucket vẫn bật Block Public Access.
- Không dùng Access Key và Secret Key trong container.

### 10. Kiểm tra Application Load Balancer

Trong AWS Management Console, truy cập:

```text
EC2 → Target Groups
```

Chọn Target Group của backend.

Kiểm tra:

- Target type là `IP`.
- Port là `3000`.
- ECS Task được đăng ký.
- Target ở trạng thái `Healthy`.
- Health check path là `/`.
- Health check trả mã `200`.

Nếu target không healthy, kiểm tra:

- Security Group.
- Container port.
- Target Group port.
- Health check path.
- Route Table.
- Backend có lắng nghe trên `0.0.0.0:3000` hay không.

### 11. Kiểm tra CloudWatch Logs

Trong AWS Management Console, truy cập:

```text
CloudWatch → Logs → Log groups → /ecs/bravelsport-backend
```

Mở log stream của ECS Task đang chạy.

Kiểm tra:

- Log startup của NestJS.
- Log API.
- Log scheduled task.
- Log kết nối MongoDB.
- Log Socket.IO nếu được ghi nhận.
- Không có secret, token hoặc connection string trong log.

<!-- CHÈN ẢNH TẠI ĐÂY:
Chụp CloudWatch Logs trong thời điểm kiểm thử.

Ảnh cần hiển thị:
- Log Group hoặc Log Stream của ECS
- Log của backend-container
- Các log event đang được ghi nhận
- Timestamp và message

Che:
- JWT
- MongoDB URI
- Password
- Email người dùng
- Account ID
- Task ID nếu không cần thiết
-->

![CloudWatch Logs trong quá trình kiểm thử](/images/5-Workshop/5.7-Testing/cloudwatch-test-logs.jpg)


## Ma trận kết quả

| Hạng mục | Kỳ vọng | Bằng chứng | Kết quả |
|---|---|---|---|
| DNS | Domain trỏ đến CloudFront | Route 53 Hosted Zone và kiểm tra DNS | PASS |
| HTTPS | Certificate hợp lệ | Truy cập `https://bravelsport.com` | PASS |
| Frontend | Trang tải và route reload thành công | `end-to-end-browser-test.jpg` | PASS |
| CloudFront | Static frontend và API đi qua distribution | DevTools Network và response headers | PASS |
| S3 Frontend | Bucket private, direct access bị từ chối | S3 Permissions và OAC Bucket Policy | PASS |
| WAF | Request hợp lệ không bị chặn | Web ACL và Sampled Requests | PASS |
| Login/JWT | Login và protected API hoạt động | DevTools Network | PASS |
| REST API | `/api/*` trả response đúng | `end-to-end-browser-test.jpg` | PASS |
| Socket.IO | `/socket.io/*` và namespace `/chat` hoạt động | Network Socket và kiểm thử chat | PASS |
| MongoDB Atlas | Backend đọc và ghi dữ liệu thành công | Atlas và API response | PASS |
| Media | Upload và hiển thị media thành công | S3 bucket `bravel-uploads` và giao diện | PASS |
| ALB | Target ECS ở trạng thái Healthy | Target Group | PASS |
| ECS Service | Desired task và running task bằng 1 | ECS Service Overview | PASS |
| CloudWatch | Backend ghi log và không lộ secret | `cloudwatch-test-logs.jpg` | PASS |

## Kiểm tra kết quả

Chương kiểm thử được xem là hoàn thành khi đáp ứng các điều kiện sau:

- Domain hoạt động qua Route 53 và CloudFront.
- HTTPS hợp lệ.
- Frontend tải thành công.
- React Router reload không lỗi.
- REST API trả response đúng.
- Authentication và JWT hoạt động.
- Socket.IO kết nối thành công.
- MongoDB Atlas đọc và ghi dữ liệu bình thường.
- Media được lưu trên Amazon S3.
- ECS Service có task đang chạy.
- Target Group ở trạng thái Healthy.
- CloudWatch Logs ghi nhận log backend.
- Không xuất hiện secret hoặc thông tin nhạy cảm trong ảnh và log.
- Không có lỗi nghiêm trọng chưa được xử lý.

## Lỗi thường gặp

| Lỗi | Nguyên nhân | Cách xử lý |
|---|---|---|
| CloudFront trả `502 Bad Gateway` | ALB không kết nối được ECS hoặc origin protocol sai | Kiểm tra ALB, Target Group, Security Group và origin protocol |
| CloudFront trả `403` | OAC, Bucket Policy hoặc WAF cấu hình sai | Kiểm tra S3 Bucket Policy, CloudFront OAC và Sampled Requests |
| React Router reload trả `403/404` | Chưa cấu hình SPA fallback | Cấu hình Custom Error Response hoặc CloudFront Function |
| API trả `403` từ WAF | Managed Rule gây false positive | Xem Sampled Requests và chuyển rule sang `Count` để phân tích |
| API trả `401` | JWT thiếu, sai hoặc hết hạn | Kiểm tra Authorization Header và logic xác thực |
| JWT không đến backend | Origin Request Policy không forward header | Forward `Authorization` đến ALB origin |
| Socket.IO polling thất bại | Cache chưa tắt hoặc thiếu query string | Dùng `CachingDisabled` và forward query string |
| WebSocket upgrade thất bại | CloudFront behavior hoặc backend cấu hình sai | Kiểm tra `/socket.io/*`, ALB và Socket.IO server |
| Namespace không kết nối | Nhầm namespace với transport path | Giữ path `/socket.io` và namespace `/chat` |
| MongoDB timeout | Atlas IP Access List hoặc NAT Gateway sai | Kiểm tra NAT EIP, Route Table và Atlas Network Access |
| Upload thất bại | ECS Task Role thiếu quyền S3 | Kiểm tra `BackendUploadS3Policy` và Task Role |
| Media mất sau redeploy | Ứng dụng vẫn lưu local `/uploads` | Chuyển hoàn toàn sang S3 Media Bucket |
| Target Group Unhealthy | Health path, port hoặc Security Group sai | Kiểm tra `/`, port `3000` và SG-ALB → SG-ECS |
| Không có log trong CloudWatch | Sai Log Group hoặc Execution Role thiếu quyền | Kiểm tra `awslogs` và `AmazonECSTaskExecutionRolePolicy` |
| Log chứa secret | Ứng dụng log environment hoặc error object nhạy cảm | Xóa log nhạy cảm, rotate secret và triển khai revision mới |

## Tóm tắt

Quá trình kiểm thử end-to-end đã xác minh toàn bộ luồng của BravelSport:

```text
User
  → Route 53
  → CloudFront
  → AWS WAF
  → S3 Frontend
```

Đối với API và Socket.IO:

```text
User
  → Route 53
  → CloudFront
  → AWS WAF
  → Application Load Balancer
  → ECS Fargate
  → MongoDB Atlas
```

Đối với media:

```text
ECS Fargate
  → Amazon S3 Media Bucket
```

CloudWatch Logs ghi nhận hoạt động của container backend. Các request API trong trình duyệt trả về mã trạng thái thành công, chứng minh frontend, backend và các dịch vụ AWS đã phối hợp đúng theo kiến trúc mục tiêu.


- Phần trước: [5.6. Database và media](../5.6-Data-and-media/)
- Phần tiếp theo: [5.8. Dọn dẹp tài nguyên](../5.8-Cleanup/)