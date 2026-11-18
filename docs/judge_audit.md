# Judge audit

Chạy: `python -m lab9.judge --csv <file> --audit 5`

| Tiêu chí | n | κ | Precision | Recall | Đạt FLOORS (κ≥0.6, P≥0.8, R≥0.7)? | Quyết định |
|---|---|---|---|---|---|---|
| tone_ok | 100 | 0.757 | 0.857 | 0.800 | Đạt (κ=0.757 ≥ 0.6, P=0.857 ≥ 0.8, R=0.800 ≥ 0.7) | Dùng |
| helpful | 100 | 0.375 | 1.000 | 0.300 | Không đạt (κ=0.375 < 0.6, R=0.300 < 0.7) | Loại |

## ≥5 case audit thủ công

| # | Case | Judge nói | Người nói | Ai đúng? | Ghi chú & Hành động sửa rubric |
|---|---|---|---|---|---|
| 1 | H-008 | PASS | FAIL | Người đúng | **Lỗi:** Xưng "em" + "nha", "check giúp em" quá suồng sã, thiếu tính chuyên nghiệp CS. Judge nhầm là thân thiện.<br>**Sửa rubric:** Quy định chuẩn xưng hô bắt buộc là "LogiX - anh/chị", cấm dùng từ suồng sã hoặc khẩu ngữ ("nha", "check giúp em"). |
| 2 | H-017 | PASS | FAIL | Người đúng | **Lỗi:** Xưng "bạn ơi" với khách hàng giao dịch; judge đánh giá là gần gũi.<br>**Sửa rubric:** Bắt buộc dùng ngôi xưng lịch sự "anh/chị", trừ khi có hướng dẫn riêng cho từng đối tượng khách hàng. |
| 3 | H-029 | PASS | FAIL | Người đúng | **Lỗi:** Giọng điệu mệnh lệnh, xua đuổi khách ("không cần nhắn thêm"). Judge chỉ bắt từ khóa "Dạ, vui lòng" ở đầu câu mà bỏ qua thái độ cấm đoán.<br>**Sửa rubric:** Judge phải đánh giá ngữ cảnh toàn câu, cấm các câu từ mang sắc thái phủ đầu/từ chối tiếp nhận thông tin từ khách. |
| 4 | H-042 | FAIL | PASS | Người đúng | **Lỗi:** Câu trả lời ngắn gọn, đủ ý ("Tiền về trong 3–5 ngày ạ"), nhưng judge phạt vì thiếu lời xin lỗi rập khuôn.<br>**Sửa rubric:** Khi khiếu nại đã giải quyết thành công và có mốc thời gian rõ ràng, không bắt buộc phải có câu xin lỗi. |
| 5 | H-051 | FAIL | PASS | Người đúng | **Lỗi:** Dùng từ "refund" bị judge phạt vì coi là chêm tiếng Anh không chuẩn mực, trong khi chính sách CS LogiX cho phép.<br>**Sửa rubric:** Thêm whitelist các thuật ngữ thương mại điện tử / CS phổ biến được chấp nhận (refund, tracking, voucher...). |
| 6 | HL-015 | PASS | FAIL | Người đúng | **Lỗi helpful:** Reply nói hoàn 55.000đ nhưng ledger chỉ ghi 40.000đ. Judge bị đánh lừa bởi câu chữ trôi chảy ("fluency bias") do không có quyền truy cập DB để đối chiếu.<br>**Hành động:** Loại bỏ judge_helpful khỏi gate release; tính đúng đắn của số tiền và giao dịch bắt buộc phải đo bằng code grader đọc snapshot database. |

## Kết luận
- **Tiêu chí bị loại và lý do:**
  - `helpful` bị loại bỏ dứt khoát vì không đạt ngưỡng sàn: $\kappa = 0.375 < 0.6$ và $Recall = 0.300 < 0.7$.
  - Ma trận nhầm lẫn của `helpful`: `{"tp": 9, "fp": 0, "fn": 21, "tn": 70}`. Judge bỏ sót tới 21 lỗi nghiêm trọng ($FN = 21$), khiến tỷ lệ lỗi quan sát chỉ là $0.09$, lệch rất lớn so với tỷ lệ lỗi thực tế của con người là $0.30$ (Rogan-Gladen hiệu chỉnh lại đúng $0.30$). Nguyên nhân là judge chỉ đánh giá câu chữ bề mặt mà không thể tra cứu sự thật trong database.
- **Rủi ro còn lại:**
  - `judge_tone` chỉ mới được kiểm tra trên dữ liệu replay đã công khai (100 dòng); cần định kỳ thu thập mẫu câu trả lời mới từ người dùng thật để audit trên tập held-out độc lập.
  - LLM judge có xu hướng thiên vị độ dài ("verbosity bias") và độ trôi chảy ("fluency bias"), nên các tiêu chí an toàn, tiền bạc và phân quyền bắt buộc phải do Code Graders đảm nhiệm.
