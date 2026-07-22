---
title: "Blog 1"
date: 2024-01-01
weight: 1
chapter: false
pre: " <b> 3.1. </b> "
---

# [AWS Security] Giải quyết tình huống biên: Chống gian lận SMS OTP với Vonage Network API và Amazon Cognito

Hôm nay tôi muốn chia sẻ bài viết về giảm thiểu gian lận mã OTP qua SMS với giải pháp dựa trên **Vonage Network API** và **Amazon Cognito**.

SMS OTP từ lâu là xương sống của nhiều hệ thống xác thực (2FA) nhờ sự tiện lợi. Nhưng đằng sau đó là nỗi đau của vận hành: gian lận cước viễn thông (**SMS Toll Fraud / AIT**) và tấn công **SIM Swap**. Attacker có thể spam OTP để đẩy billing, hoặc chiếm số điện thoại để bypass lớp bảo mật.

Dưới đây là những điểm thực tế rút từ kiến trúc AWS + Vonage để xử lý vấn đề này từ gốc.

## 1. Vì sao chỉ chặn IP / Rate Limit là chưa đủ?

Khi hệ thống bị spam OTP, phản xạ thường gặp là Rate Limit hoặc chặn IP trên **AWS WAF**. Tuy nhiên botnet hiện đại dùng nhiều IP “sạch” và giả lập hành vi người dùng thật.

Cái còn thiếu không chỉ là kiểm soát hạ tầng web, mà là tín hiệu từ nhà mạng: **số điện thoại này có an toàn trước khi gửi OTP không?**

## 2. Công nghệ nền tảng: Network APIs

Giải pháp tích hợp Network APIs theo chuẩn **CAMARA** (Vonage / Ericsson) vào luồng xác thực AWS. Hai API đáng chú ý:

- **SIM Swap API:** kiểm tra ngầm với nhà mạng xem SIM vừa bị đổi gần đây (thường 24–72 giờ) hay không.
- **Number Verification API:** xác thực thẻ SIM đang dùng data di động có khớp số điện thoại khai báo hay không — silent authentication, ít ma sát hơn OTP thủ công.

## 3. Kiến trúc: Amazon Cognito + Vonage

Logic kiểm tra của Vonage được đưa vào luồng **Amazon Cognito** qua **Lambda Triggers**.

![Kiến trúc Risk-Adaptive Customer Sign-In](/fcj-internship-report/images/3-BlogsPosted/blog1.png)

> Nguồn bài: [AWS Study Group — Facebook](https://www.facebook.com/groups/awsstudygroupfcj/permalink/2198070834291210/)

### Luồng xử lý (Lớp 1 → 6)

1. **User & Edge:** Request từ Web/iOS/Android đi qua **CloudFront + AWS WAF** (rate limit, bot blocking, DDoS).
2. **Front door:** Traffic sạch tới **API Gateway + Auth Service (BFF)** để điều phối đăng nhập.
3. **Identity + Risk:** **Amazon Cognito + Risk Engine** đánh giá rủi ro LOW / MEDIUM / HIGH.
4. **Verification:**
   - Ưu tiên **Passkey / FIDO2** khi thiết bị hỗ trợ.
   - Hoặc gọi Vonage (**Identity Insights**, **Verify API**, **Fraud Defender**) và tín hiệu nhà mạng (SIM-swap, Silent Auth, SMS/Voice).

## 4. Bài học áp dụng được

- **Phòng thủ chủ động:** chặn sớm trong luồng auth thay vì đợi billing SMS tăng rồi mới xử lý.
- **UX tốt hơn:** Number Verification giúp giảm drop-rate khi đăng ký/đăng nhập.
- **Zero-trust ở lớp viễn thông:** không tin tuyệt đối vào client hay số hiển thị; xác thực thêm với nhà mạng.

Tôi thấy đây là hướng đáng tham khảo khi hệ thống CIAM cần vừa bảo mật vừa giữ trải nghiệm mượt.
