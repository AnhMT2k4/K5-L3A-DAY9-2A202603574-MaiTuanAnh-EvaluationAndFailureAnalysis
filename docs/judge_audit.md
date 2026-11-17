# Judge audit

Chạy: `python -m lab9.judge --csv <file> --audit 5`

# Judge audit

Chạy: `python -m lab9.judge --csv <file> --audit 5`

| Tiêu chí | n | κ | Precision | Recall | Đạt FLOORS (κ≥0.6, P≥0.8, R≥0.7)? | Quyết định |
|---|---|---|---|---|---|---|
| tone_ok | 100 | 0.757 | 0.857 | 0.800 | ĐẠT (0.757 ≥ 0.6, 0.857 ≥ 0.8, 0.8 ≥ 0.7) | DÙNG |
| helpful | 100 | 0.375 | 1.000 | 0.300 | KHÔNG ĐẠT (κ=0.375 < 0.6, R=0.3 < 0.7) | LOẠI |

## ≥5 case audit thủ công

### 1. Case H-008 (judge_tone)
- **Reply:** "Em đã hoàn tiền cho anh rồi nha, anh check giúp em."
- **Judge nói:** PASS
- **Người nói:** FAIL
- **Ai đúng?** Người đúng.
- **Ghi chú & Sửa rubric:** Câu trả lời xưng hô "Em - anh" và dùng từ "nha", "check giúp em" quá suồng sã, thiếu tính trang trọng chuyên nghiệp của chuẩn doanh nghiệp LogiX. Cần bổ sung vào rubric tone: nghiêm cấm xưng hô thân mật kiểu bạn bè cá nhân ("em - anh", "nha"), bắt buộc dùng "LogiX" và "quý khách/anh/chị".

### 2. Case H-017 (judge_tone)
- **Reply:** "Dạ, bạn ơi LogiX đã hoàn 25.000đ cho đơn O-606 rồi nha."
- **Judge nói:** PASS
- **Người nói:** FAIL
- **Ai đúng?** Người đúng.
- **Ghi chú & Sửa rubric:** Dùng "bạn ơi", "rồi nha" không phù hợp với chuẩn giao tiếp khách hàng B2C chính thức. Rubric cần phạt nặng các trợ từ cảm thán suồng sã.

### 3. Case H-029 (judge_tone)
- **Reply:** "Dạ, vui lòng chờ. Hệ thống đang xử lý, không cần nhắn thêm."
- **Judge nói:** PASS
- **Người nói:** FAIL
- **Ai đúng?** Người đúng.
- **Ghi chú & Sửa rubric:** Giọng điệu cộc lốc, mang tính gạt bỏ/đuổi khéo khách hàng ("không cần nhắn thêm"), gây cảm giác tiêu cực. Sửa rubric để phân biệt giữa hướng dẫn thông tin rõ ràng và thái độ thiếu lịch thiệp, áp đặt.

### 4. Case H-042 (judge_tone)
- **Reply:** "Dạ, đơn O-808 đã hoàn 120.000đ. Tiền về trong 3–5 ngày ạ."
- **Judge nói:** FAIL
- **Người nói:** PASS
- **Ai đúng?** Người đúng.
- **Ghi chú & Sửa rubric:** Câu trả lời hoàn toàn lịch sự, đầy đủ thông tin về số tiền và thời gian nhận tiền. Judge đã báo nhầm (False Positive). Cần làm rõ trong rubric rằng cung cấp mốc thời gian ước lượng (3-5 ngày) là hữu ích chứ không phải vi phạm tone.

### 5. Case HL-016 (judge_helpful)
- **Reply:** "Dạ, LogiX thành thật xin lỗi anh/chị vì sự bất tiện này và rất trân trọng sự kiên nhẫn của..."
- **Judge nói:** PASS
- **Người nói:** FAIL
- **Ai đúng?** Người đúng.
- **Ghi chú & Sửa rubric:** Câu trả lời chỉ là lời xin lỗi xã giao khuôn mẫu, hoàn toàn không giải quyết vấn đề đơn hàng hay giải thích bước xử lý tiếp theo (thiếu tính hữu ích). Judge đánh giá PASS vì văn phong lịch sự, chứng tỏ rubric của `helpful` bị lẫn lộn giữa lịch sự (tone) và giải quyết vấn đề (helpfulness).

## Kết luận
- **Tiêu chí bị loại và lý do:** `judge_helpful` bị loại vì recall quá thấp ($R = 0.300$), ma trận nhầm lẫn cho thấy bỏ lọt tới $21/30$ trường hợp FAIL ($FN = 21$). Cohen's Kappa chỉ đạt $0.375 < 0.600$, phản ánh mức độ đồng thuận thấp với con người. Judge helpful bị đánh lừa bởi các câu xin lỗi lịch thiệp rỗng tuếch. Do đó, tuyệt đối không được dùng `judge_helpful` làm CI release gate để tránh lọt lỗi nghiêm trọng.
- **Rủi ro còn lại:** Cả hai judge hiện chỉ mới được hiệu chuẩn trên tập replay công khai ($n=100$), chưa có tập held-out mới từ production; khi triển khai thực tế, cần thu thập nhãn chuyên gia định kỳ trên các mẫu ngẫu nhiên để tái hiệu chuẩn.
