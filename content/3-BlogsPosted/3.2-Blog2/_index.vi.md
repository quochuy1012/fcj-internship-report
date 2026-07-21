---
title: "Blog 2"
date: 2024-01-01
weight: 2
chapter: false
pre: " <b> 3.2. </b> "
---

# Đơn giản hóa truy cập dữ liệu doanh nghiệp với AWS Transfer Family Web Apps và Terraform

Trong nhiều doanh nghiệp, dữ liệu quan trọng thường nằm trên **Amazon S3**. Vấn đề thực tế là không phải ai cũng dùng được AWS Console, CLI hay tool kỹ thuật để lấy file.

Làm sao để nhân viên upload/download trên S3 một cách đơn giản, an toàn và dễ quản trị?

Một hướng phù hợp là: **AWS Transfer Family Web Apps** kết hợp **IAM Identity Center**, **Amazon S3 Access Grants**, **Amazon S3**, **AWS CloudTrail** và **Terraform**. Transfer Family Web Apps cung cấp cổng web để user đã xác thực duyệt, tải lên và tải xuống dữ liệu S3 mà không cần vào Console hay tự dựng portal phức tạp.

## Kiến trúc tổng quan

![Kiến trúc AWS Transfer Family Web Apps](/images/3-BlogsPosted/blog2.png)

> Nguồn bài: [AWS Study Group — Facebook](https://www.facebook.com/photo/?fbid=1533670831545926)

Luồng chính gồm 5 bước:

1. **End User xác thực với IAM Identity Center** — có thể tích hợp External IdP (Azure AD, Okta, SAML…).
2. **Sau khi đăng nhập, user vào Transfer Family Web Apps** — giao diện web thân thiện, thao tác file như portal nội bộ.
3. **Transfer Family Web Apps lấy quyền từ S3 Access Grants** — phân quyền chi tiết theo user/group thay vì cấp quyền quá rộng.
4. **User upload/download object trên Amazon S3** — theo đúng quyền đã được cấp.
5. **CloudTrail ghi log hoạt động** — phục vụ audit, compliance và điều tra sự cố.

## Vai trò của Terraform

Terraform giúp triển khai hạ tầng theo hướng Infrastructure as Code: đóng gói Identity Center, S3, Access Grants, CloudTrail và Transfer Family Web Apps thành deployment lặp lại được cho Dev/Staging/Prod hoặc nhiều phòng ban.

## Giá trị bảo mật và vận hành

- User không cần vào AWS Console
- Không phải tự xây custom portal từ đầu
- Phân quyền theo user/group rõ ràng
- Có audit log
- Tích hợp hệ thống định danh sẵn có
- Triển khai lặp lại bằng Terraform

## Kết luận

Transfer Family Web Apps giúp biến S3 thành cổng truy cập dữ liệu thân thiện hơn cho người dùng doanh nghiệp. Kết hợp Identity Center, Access Grants, CloudTrail và Terraform thì giải pháp vừa đơn giản phía user, vừa kiểm soát được phía IT.
