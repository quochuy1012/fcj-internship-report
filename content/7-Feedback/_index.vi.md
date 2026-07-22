---
title: "Chia sẻ, đóng góp ý kiến"
date: 2024-01-01
weight: 7
chapter: false
pre: " <b> 7. </b> "
---

Kỳ thực tập **First Cloud AI Journey** tại **AWS Viet Nam** (15/05/2026 – 31/07/2026) với tôi không phải lúc nào cũng suôn sẻ, nhưng đúng là học được nhiều. Phần dưới đây tôi viết thẳng theo những gì đã trải qua, mong team FCAJ lấy làm tham khảo cho các kỳ sau.

### Đánh giá chung

**1. Môi trường làm việc**  
Ban đầu tôi hơi e vì không biết hỏi ai, hỏi lúc nào cho hợp. Sau vài tuần thì quen hơn: group luôn có người trả lời, tài liệu cũng đủ để tự bắt đầu. Cảm giác chung là thoải mái, không bị “soi” từng bước. Điểm tôi vẫn thấy thiếu là ít buổi để quen mặt nhau ngoài chuyện kỹ thuật. Có tuần tôi muốn hỏi nhanh một lỗi ECS mà ngại vì chưa thân ai trong group.

**2. Sự hỗ trợ của mentor / team admin**  
Mentor sẵn sàng hỗ trợ, nhưng không làm thay. Khi tôi kẹt, họ thường hỏi lại tôi đã thử gì rồi mới gợi ý bước tiếp theo — cách này giúp tôi nhớ lâu hơn là chỉ nhận đáp án. Lúc review kiến trúc **BravelSport**, góp ý khá cụ thể: diagram cần rõ hơn, nghĩ thêm tính sẵn sàng và phần kết nối backend ra ngoài VPC. Team admin hỗ trợ lịch học và thủ tục nhanh, tôi không phải chờ lâu khi cần giấy tờ hay xác nhận.

**3. Sự phù hợp với chuyên ngành**  
Tôi học CNTT tại HUTECH, nên kiến thức mạng và hệ thống vẫn dùng được khi làm lab AWS. Phần mới nhất với tôi là triển khai cloud end-to-end: không chỉ học từng dịch vụ riêng lẻ mà phải gắn chúng vào một dự án thật. Với **BravelSport**, tôi hiểu hơn việc thiết kế frontend, backend, lưu trữ và giám sát phải đi cùng nhau thì hệ thống mới chạy ổn.

**4. Cơ hội học hỏi**  
Tôi học nhiều nhất không phải lúc làm lab lần đầu, mà lúc phải sửa lại sau góp ý và ngồi viết proposal/workshop. Community Day cũng mở mắt: nghe người khác kể cách họ làm sản phẩm, tôi mới thấy cloud rộng hơn cái Console trên máy mình. Phần khó nhất với tôi vẫn là giải thích kiến trúc cho người khác nghe — làm được rồi nhưng nói chưa gọn.

**5. Văn hóa & đồng đội**  
Trong FCAJ mọi người khá sẵn sàng chia sẻ. Tôi hỏi mấy lần về lỗi deploy mà không bị “ngó lơ”. So với tự học trên YouTube thì đỡ cô đơn hơn nhiều. Community Day cũng là lúc tôi cảm thấy mình thuộc một cộng đồng thật, không chỉ là người làm bài một mình.

**6. Chính sách / phúc lợi**  
Chương trình có lộ trình rõ từ đầu nên tôi biết mình đang ở giai đoạn nào và cần nộp gì. Điều tôi đánh giá cao là khi lab và viết tài liệu bị dồn cùng lúc, lịch vẫn linh hoạt được một phần — nhờ vậy tôi kịp hoàn thành báo cáo mà không phải bỏ dở phần thực hành.

### Câu hỏi khác

- **Hài lòng nhất:** Nhìn **BravelSport** từ chỗ chỉ là ý tưởng và vài service rời, sau vài vòng feedback dần thành một kiến trúc mình giải thích được. Cảm giác đó đáng hơn lúc “lab pass”.
- **Nên cải thiện:** Nên review kiến trúc sớm hơn, đừng để gần cuối mới phát hiện diagram còn lab-like. Thêm buổi ngắn về lỗi hay gặp trên ECS / networking cũng sẽ giúp nhiều bạn đỡ mất thời gian mò.
- **Có giới thiệu bạn bè không?** Có, nhưng tôi sẽ nói trước: chương trình này cần chịu ngồi lab và viết tài liệu. Ai chỉ muốn “đi cho có” thì sẽ thấy nặng; ai chịu làm thì học được thứ mang đi dùng được.

### Đề xuất & mong muốn

- Ngay từ vòng proposal đầu, cho một checklist ngắn (diagram high-level, Multi-AZ, private subnet, kết nối DB, monitoring, cleanup) để đỡ sửa lớn về sau.
- Nếu được, thêm peer review nhẹ giữa các bạn cùng kỳ — nhìn diagram của người khác cũng học được nhiều.
- Sau bootcamp này tôi muốn đi sâu hơn phần vận hành và tối ưu chi phí, không dừng ở mức dựng được hệ thống.
- Cảm ơn mentor và team FCAJ đã kèm trong kỳ thực tập.
