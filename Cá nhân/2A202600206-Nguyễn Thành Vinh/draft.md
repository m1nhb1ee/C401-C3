# TOP 3 Failure Mode

## 1. Ambiguous Query & Generic Response (Lỗi phản hồi vô định hướng)
Học viên mô tả lỗi quá sơ sài ("Code em không chạy", "Em không hiểu bài này"), dẫn đến AI đưa ra các lời khuyên sáo rỗng
- **Trigger:** Học viên thiếu kỹ năng đặt câu hỏi (Prompt Engineering) hoặc không biết cách cung cấp ngữ cảnh (như báo lỗi, ảnh chụp màn hình, đoạn code cụ thể).
- **Hậu quả:** Học viên nhận được các gợi ý chung chung (ví dụ: "Hãy kiểm tra lại cú pháp"). Họ cảm thấy AI "ngu" và vô dụng, dẫn đến việc bỏ rơi sản phẩm và tăng tỉ lệ rời bỏ khóa học (churn rate).
- **Mitigation:** Thay vì trả lời ngay, thiết lập để AI tự động hỏi lại các câu hỏi dẫn dắt: "Bạn có thể gửi đoạn code hoặc thông báo lỗi cụ thể không?" hoặc "Bạn đang gặp khó khăn ở dòng code nào?". **Prompt Templates**: Cung cấp các nút bấm gợi ý hoặc khung nhập liệu có sẵn (ví dụ: "Tôi gặp lỗi khi chạy...", "Giải thích giúp tôi khái niệm...") để định hướng học viên.

## 2. Pedagogy Bypass (Lỗi vượt rào sư phạm)
Học viên dùng Chatbot như một công cụ "giải hộ" để lấy đáp án nhanh thay vì để học.
- **Trigger:** Học viên lười tư duy hoặc đang chịu áp lực về thời hạn nộp bài. Họ copy-paste nguyên đề bài và yêu cầu AI đưa ra lời giải/đáp án cuối cùng.
- **Hậu quả:** Sinh viên vượt qua bài kiểm tra nhưng không nắm vững kiến thức. Điều này làm hỏng giá trị cốt lõi của giáo dục và khiến nhà trường/giảng viên ác cảm, có thể dẫn đến việc cấm sử dụng AI trong học tập.
- **Mitigation:** Cấu hình hệ thống để AI không bao giờ đưa ra đáp án trực tiếp. Thay vào đó, AI sẽ chỉ ra lỗ hổng trong logic của học viên hoặc gợi ý các bước (step-by-step) để học viên tự giải.

## 3. Knowledge Stale/Drift (Lỗi lạc hậu dữ liệu)
Tài liệu khóa học đã thay đổi (cập nhật thư viện mới, quy trình mới) nhưng Vector Database của RAG vẫn giữ thông tin cũ.
- **Trigger:** Quy trình cập nhật nội dung khóa học từ bộ phận học thuật và quy trình cập nhật dữ liệu vào AI (Tech Team) không đồng bộ. Tài liệu mới được tải lên Web nhưng chưa được Re-index vào hệ thống RAG.
- **Hậu quả:** AI hướng dẫn sinh viên dùng các hàm cũ đã bị khai tử hoặc làm bài theo quy chuẩn cũ. Sinh viên làm theo và bị chấm điểm sai, gây ra sự phẫn nộ và mất niềm tin vào hệ thống của nhà trường.
- **Mitigation:** Gắn nhãn thời gian cho từng đoạn kiến thức. Khi AI trả lời, nó phải hiển thị: "Dựa trên tài liệu cập nhật ngày DD/MM/YYYY" để người dùng tự đối chiếu.