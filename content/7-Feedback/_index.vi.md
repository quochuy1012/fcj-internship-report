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
Mentor không làm hộ; thường bảo tôi thử trước rồi mới chỉ hướng. Lúc đầu hơi sốt ruột, nhưng về sau tôi thấy cách này hợp. Lần review **BravelSport**, mentor nói diagram còn rối và chưa nghĩ tới Multi-AZ / outbound ra Atlas — nghe hơi “đắng” nhưng đúng. Admin thì lo lịch và giấy tờ ổn, gần như không phải nhắc hai lần.

**3. Sự phù hợp với chuyên ngành**  
Học CNTT ở HUTECH nên phần mạng, hệ thống tôi không bị lạc quá. Cái mới là cloud: trước chỉ biết tên dịch vụ, kỳ này mới phải ghép chúng thành một hệ thống chạy được. Dự án **BravelSport** (`bravelsport.com`) giúp tôi thấy CloudFront, ECS, ALB, S3 không còn là slide nữa mà gắn với luồng bán hàng và đặt sân thật.

**4. Cơ hội học hỏi**  
Tôi học nhiều nhất không phải lúc làm lab lần đầu, mà lúc phải sửa lại sau góp ý và ngồi viết proposal/workshop. Community Day cũng mở mắt: nghe người khác kể cách họ làm sản phẩm, mình mới thấy cloud rộng hơn cái Console trên máy mình. Phần khó nhất với tôi vẫn là giải thích kiến trúc cho người khác nghe — làm được rồi nhưng nói chưa gọn.

**5. Văn hóa & đồng đội**  
Trong FCAJ mọi người khá sẵn sàng chia sẻ. Tôi hỏi mấy lần về lỗi deploy mà không bị “ngó lơ”. So với tự học trên YouTube thì đỡ cô đơn hơn nhiều. Community Day cũng là lúc tôi cảm thấy mình thuộc một cộng đồng thật, không chỉ là người làm bài một mình.

**6. Chính sách / phúc lợi**  
Lộ trình khá rõ: worklog, proposal, event, workshop rồi báo cáo. Nhờ vậy tôi biết tuần nào nên dồn sức vào đâu. Có lúc deadline tài liệu chồng lên lab thì được linh hoạt thời gian — nếu không có điểm này chắc tôi đã rối hơn.

### Câu hỏi khác

- **Hài lòng nhất:** Nhìn **BravelSport** từ chỗ chỉ là ý tưởng và vài service rời, sau vài vòng feedback dần thành một kiến trúc mình giải thích được. Cảm giác đó đáng hơn lúc “lab pass”.
- **Nên cải thiện:** Nên review kiến trúc sớm hơn, đừng để gần cuối mới phát hiện diagram còn lab-like. Thêm buổi ngắn về lỗi hay gặp trên ECS / networking cũng sẽ giúp nhiều bạn đỡ mất thời gian mò.
- **Có giới thiệu bạn bè không?** Có, nhưng tôi sẽ nói trước: chương trình này cần chịu ngồi lab và viết tài liệu. Ai chỉ muốn “đi cho có” thì sẽ thấy nặng; ai chịu làm thì học được thứ mang đi dùng được.

### Đề xuất & mong muốn

- Ngay từ vòng proposal đầu, cho một checklist ngắn (diagram high-level, Multi-AZ, private subnet, kết nối DB, monitoring, cleanup) để đỡ sửa lớn về sau.
- Nếu được, thêm peer review nhẹ giữa các bạn cùng kỳ — nhìn diagram của người khác cũng học được nhiều.
- Sau bootcamp này tôi muốn đi sâu hơn phần vận hành và tối ưu chi phí, không dừng ở mức dựng được hệ thống.
- Cảm ơn mentor và team FCAJ đã kèm trong kỳ thực tập.
