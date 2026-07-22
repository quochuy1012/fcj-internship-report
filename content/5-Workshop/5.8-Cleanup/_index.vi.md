---
title: "Dọn dẹp tài nguyên"
date: 2026-07-19
weight: 8
chapter: false
pre: " <b> 5.8. </b> "
---


# Dọn dẹp tài nguyên

## Mục tiêu

Xóa tài nguyên workshop theo thứ tự giảm dependency và tránh tiếp tục phát sinh chi phí.

### Lưu ý

Không xóa Hosted Zone, domain, CloudFront distribution, bucket, database hoặc record đang phục vụ website thật khi chưa có backup, kế hoạch chuyển đổi và phê duyệt. Trước mỗi thao tác, xác nhận Account, Region, Project và Environment.

## Các bước thực hiện

Thứ tự đề xuất:

1. Xóa Route 53 record chỉ dùng cho workshop.
2. Disable và xóa CloudFront Distribution.
3. Gỡ WAF association và xóa Web ACL/rule chỉ dùng cho workshop.
4. Scale ECS Service về `0` nếu cần quan sát trước.
5. Xóa ECS Service.
6. Xác nhận ECS Task đã dừng.
7. Xóa ALB.
8. Xóa Target Group.
9. Xóa ECS Cluster nếu không còn service/task.
10. Deregister hoặc xóa Task Definition revision nếu cần.
11. Xóa ECR image và repository.
12. Xóa CloudWatch Log Group.
13. Làm rỗng và xóa S3 Frontend Bucket.
14. Làm rỗng và xóa S3 Media Bucket.
15. Xóa NAT Gateway và chờ trạng thái `Deleted`.
16. Release Elastic IP sau khi NAT đã xóa.
17. Terminate EC2 Build Machine và kiểm tra EBS volume/snapshot.
18. Xóa route table custom sau khi gỡ association.
19. Xóa Security Group sau khi không còn ENI phụ thuộc.
20. Xóa subnet.
21. Detach và xóa Internet Gateway.
22. Xóa VPC.
23. Xóa IAM Role/Policy chỉ dùng cho workshop sau khi xác nhận không còn resource tham chiếu.
24. Kiểm tra Route 53 Hosted Zone/domain registration và Atlas; chỉ xóa nếu thực sự thuộc phạm vi workshop.

<!-- CHÈN ẢNH TẠI ĐÂY:
Chụp NAT Gateway ở trạng thái Deleting/Deleted. Che NAT ID và Elastic IP.
Thông tin cần che: Account ID, ARN, địa chỉ IP công khai, token, cookie, secret và dữ liệu người dùng nếu xuất hiện.
-->

<!-- Ảnh chưa có trong static: /fcj-internship-report/images/5-Workshop/5.8-Cleanup/delete-nat-gateway.png -->
<!-- CHÈN ẢNH TẠI ĐÂY:
Chụp EC2 Build Machine trạng thái Terminated. Che Instance ID và IP.
Thông tin cần che: Account ID, ARN, địa chỉ IP công khai, token, cookie, secret và dữ liệu người dùng nếu xuất hiện.
-->

<!-- Ảnh chưa có trong static: /fcj-internship-report/images/5-Workshop/5.8-Cleanup/ec2-terminated.png -->

## Kiểm tra kết quả

- Không còn ECS service/task chạy.
- Không còn ALB/NAT Gateway.
- Elastic IP đã release.
- S3 bucket workshop đã xóa hoặc được giữ có lý do.
- EC2 đã terminate và EBS không còn ngoài kế hoạch.
- VPC resource map không còn dependency.
- IAM role workshop không còn được dùng.
- Cost Explorer/Billing không còn tài nguyên chạy ngoài kế hoạch sau thời gian cập nhật dữ liệu.

<!-- CHÈN ẢNH TẠI ĐÂY:
Chụp VPC resource map hoặc Resource Explorer không còn tài nguyên workshop; có thể thêm Cost Explorer sau khi dữ liệu cập nhật. Che Account ID và chi phí nhạy cảm.
Thông tin cần che: Account ID, ARN, địa chỉ IP công khai, token, cookie, secret và dữ liệu người dùng nếu xuất hiện.
-->

<!-- Ảnh chưa có trong static: /fcj-internship-report/images/5-Workshop/5.8-Cleanup/verify-cleanup.png -->

## Lỗi thường gặp

| Lỗi | Nguyên nhân | Cách xử lý |
|---|---|---|
| Không xóa VPC | Còn ENI, NAT, ALB hoặc endpoint | Kiểm tra Network Interfaces và dependency |
| Không xóa SG | Còn ENI hoặc SG reference | Xóa resource phụ thuộc trước |
| Không xóa bucket | Còn object/version/delete marker | Làm rỗng tất cả version |
| EIP vẫn tính phí | Chưa release sau khi xóa NAT | Release Elastic IP |
| CloudFront chưa xóa | Distribution chưa disabled/deployed | Disable, chờ deployed rồi delete |

## Tóm tắt

Tài nguyên workshop đã được xóa theo thứ tự an toàn và chi phí tồn dư đã được rà soát.

## Danh sách ảnh

| STT | Tên file | Nội dung ảnh | Vị trí chèn |
|---:|---|---|---|
| 1 | `delete-nat-gateway.png` | NAT Gateway đã chuyển sang Deleted | Sau bước xóa NAT |
| 2 | `verify-cleanup.png` | VPC/resource list và Billing sau clean-up | Cuối chương |
| 3 | `ec2-terminated.png` | EC2 Build Machine terminated | Sau bước xóa EC2 |

## Điều hướng

- Phần trước: [5.7. Kiểm thử end-to-end](../5.7-Testing/)
- Phần tiếp theo: [Workshop](../)
