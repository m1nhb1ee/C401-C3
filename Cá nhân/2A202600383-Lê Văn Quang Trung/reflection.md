# Reflection Cá Nhân - Hackathon Project
**Dự án:** TA ChatBot

**Thành viên:** Lê Văn Quang Trung

---

### 1. Vai trò cụ thể trong nhóm
- **Chuyên môn kỹ thuật (Technical / Product Development):** Chịu trách nhiệm chính trong việc phát triển sản phẩm, biến ý tưởng thành hệ thống demo có thể hoạt động được.

    Link GitHub Chat_Bot: https://github.com/kagamikuro1024/TA_ChatBot

### 2. Phần phụ trách cụ thể
Tôi phụ trách mảng Coding và Triển khai (Deploy) sản phẩm dựa trên bản thiết kế (SPEC) được nhóm chuẩn bị. Cụ thể, các công việc có output rõ ràng bao gồm:
- **Thiết kế xây dựng Agent Architecture (LangGraph):** 
    Triển khai luồng xử lý tổng thể của Agent, sử dụng `create_react_agent` để liên kết mô hình ngôn ngữ lớn (LLM) với bộ công cụ tùy chỉnh, đảm bảo khả năng lên kế hoạch và gọi đúng tool dựa trên ý định của sinh viên.
- **Phát triển AI Tools & RAG:** 
    Lập trình các công cụ cốt lõi cho trợ giảng ảo, bao gồm: `search_materials` (truy xuất tài liệu học tập qua FAISS vector store), `code_analyzer` (hướng dẫn debug/sửa lỗi code cho người học), và `course_info` (truy vấn nhanh thông tin/chính sách khóa học).
- **Xây dựng Hệ thống Chuyển tiếp (Escalation System):** 
    Lập trình module `detect_trigger` để nhận diện các tình huống sinh viên nhầm lẫn, tranh cãi hoặc bất đồng. Cùng với đó là module `escalation` kết hợp hệ thống tự động gửi Email báo cáo về cho các TA / giảng viên thật xử lý khi AI không thể giải quyết.
- **Triển khai Kiểm chứng & Tránh ảo giác (Information Verification):** Xây dựng module `verify_information` kết hợp chặt chẽ vào quy trình suy luận (reasoning) nhằm kiểm tra chéo thông tin. Điều này giúp ngăn chặn AI tự "bịa" đáp án (hallucinate) ngoài phạm vi tri thức của môn học.

### 3. Đánh giá SPEC (Thiết kế)
- **Phần mạnh nhất:** 
    Luồng kiến trúc Agent và bộ công cụ AI tự động. 
    Vì nhóm đã thiết kế kỹ lưỡng rõ về LangGraph cũng như các công cụ cần thiết (RAG, Escalation, Verification), nên việc kết nối và định tuyến các node diễn ra khá suôn sẻ và có hệ thống.
- **Phần yếu nhất:** 
    Việc xử lý các ngoại lệ (Edge cases) và kịch bản đặc biệt (Special scenarios). 
    Do SPEC ban đầu tập trung nhiều vào 'happy path' (luồng lý tưởng) mà chưa lường trước các tình huống hội thoại phức tạp của người dùng, dẫn đến lúc code thực tế và ráp nối dễ bị kẹt logic.

### 4. Đóng góp cụ thể khác (Ngoài Coding)
Bên cạnh công việc lập trình, tôi còn đóng góp vào các khâu khác của dự án:
- **Kiểm thử và Debug:** Trực tiếp lên các *test scenarios*, thực hiện quy trình kiểm thử chất lượng chatbot và gỡ lỗi (debug) xuyên suốt quá trình phát triển.
- **Thiết kế Demo và Thuyết trình:** Phụ trách thiết kế bản demo cuối cùng của sản phẩm để trình diễn, đồng thời làm người đại diện thuyết trình bảo vệ sản phẩm trước các nhóm.

### 5. Một điều học được trong hackathon
- **Cách thiết kế SPEC:** Trước đây tôi thường code trực tiếp, thông qua hackathon tôi đã học được cách lên bản thiết kế phần mềm (SPEC) một cách bài bản trước khi triển khai, hiểu được tầm quan trọng của việc có thiết kế rõ ràng đối với tiến độ và độ chính xác của dự án.

### 6. Nếu làm lại, sẽ đổi gì?
- Nếu được làm lại, tôi sẽ **thiết kế plan/bản vẽ luồng dữ liệu thật kỹ lưỡng và chi tiết trước khi nhờ Agent (AI) hỗ trợ "vibe code"**. Việc này sẽ giúp rào trước các bối cảnh cụ thể và kịch bản bất thường (special scenarios), từ đó đỡ mất rất nhiều thời gian ngồi cắt nghĩa lại luồng và debug những đoạn code AI sinh ra sai ý đồ.

### 7. Trải nghiệm làm việc với AI
- **AI giúp gì:** Hỗ trợ sinh code cực kì nhanh chóng đối với các hàm, thuật toán hay block chức năng cơ bản. Nó đóng vai trò như một người "thợ gõ" giúp tiết kiệm thời gian gõ boilerplate code hay triển khai nhanh các cấu trúc LangGraph đã định hình.
- **AI sai/mislead ở đâu:** AI thường sai lầm và gây hiểu lầm nặng khi context dự án rẽ vào các bối cảnh quá đặc thù (special scenarios). Nếu không có design plan/bản vẽ rõ ràng mà để cho AI tự vibe code, nó có xu hướng tự biên dịch sai mục đích, sinh ra các luồng logic lặp vô tận hoặc dùng sai thư viện, dẫn tới việc developer phải tốn thời gian đọc hiểu và debug.
