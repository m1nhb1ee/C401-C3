# Individual reflection — Hoàng Đức Nghĩa (2A202600371)

## 1. Role
*Phụ trách phân tích problem, thiết kế solution ở mức high-level và hoàn thiện AI Product Canvas

## 2. Đóng góp cụ thể
*Xác định rõ problem thực tế: TA overload (50–100 câu/ngày, 60–70% FAQ lặp lại)
*Thiết kế Value Proposition cho 2 nhóm:
- Student: giảm thời gian chờ từ hours → seconds
- TA: tiết kiệm 3–5h/ngày
*Phân tách rõ 3 loại use case:
- FAQ → Auto
- Technical → Augment
- Ambiguous → Escalate
*Xây dựng phần Trust:
- Failure modes (AI trả lời sai, hướng sai)
- Mitigation: confidence threshold, fallback “Hỏi TA thật”, audit hàng tuần
*Phân tích Feasibility:
- Latency, cost per query (~$0.02–0.05)
- Tech stack (RAG + vector DB)
*Thiết kế Learning Signal loop:
- Đánh giá từ student
- Correction từ TA → tạo dataset cải tiến
- Weekly metrics + error pattern

## 3. SPEC mạnh/yếu
*Mạnh nhất: phân tách rõ automation vs augmentation
Không cố gắng auto toàn bộ mà thiết kế hệ thống hybrid:
- Giảm rủi ro AI sai
- Vẫn tối ưu được hiệu suất TA
→ Đây là quyết định product quan trọng, không phải technical
*Yếu nhất: một số assumption chưa đủ rõ
- Latency (<3s, <10s) chưa có benchmark thực tế
- Escalation rate chưa có baseline từ data thật
- ROI chưa tách rõ theo từng kịch bản sử dụng

→ Canvas tốt về logic nhưng còn thiếu validation bằng data

## 4. Đóng góp khác
*Đề xuất idea data flywheel:
- Error pattern theo tuần (VD: lỗi phổ biến tuần 3 Python)
- Insight cho giảng viên → cải thiện curriculum
*Góp phần định nghĩa metric hệ thống:
- Auto-response rate
- False positive/negative escalation
- Student “hữu ích” rate

## 5. Điều học được
Trước hackathon nghĩ nghĩ AI product = model + prompt.
Sau khi làm canvas, hiểu rằng: Phần quan trọng nhất là thiết kế hệ thống và risk control.
Cụ thể:
- Không phải câu nào cũng nên auto
- Phải xác định khi nào AI nên im lặng và nhường cho người
- Trust và fallback quan trọng không kém accuracy
Ngoài ra học được: Value thật sự đến từ việc giải quyết pain cụ thể, không phải thêm AI vào cho “ngầu”.

## 6. Nếu làm lại
*Sẽ validate sớm bằng data thực tế từ TA (log câu hỏi thật)
*Làm rõ hơn các assumption:
- Escalation rate
- Distribution câu hỏi (FAQ vs technical)
- Bổ sung thiết kế cho input quality (vì user thường mô tả lỗi mơ hồ)

## 7. AI giúp gì / AI sai gì
**Giúp:** 
- Brainstorm nhanh các failure modes và edge cases
- Gợi ý structure cho AI Product Canvas (Value – Trust – Feasibility)
- Hỗ trợ refine wording để rõ ràng và logic hơn
**Sai/mislead:** 
- Đôi khi đề xuất solution quá “ideal” (auto nhiều hơn mức an toàn)
- Gợi ý thêm feature ngoài scope (VD: mở rộng sang full learning platform)
