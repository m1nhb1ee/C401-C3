# Báo Cáo: Trợ Thủ Tài Chính Moni của MoMo

**Ngày báo cáo:** April 9, 2026  
**Chủ đề:** Phân tích trải nghiệm người dùng với AI Assistant Moni  
**Tập trung:** Phân tích 4 luồng xử lý, đánh giá điểm mạnh/yếu, và đề xuất cải thiện

---

## 1. Giải Thích Kết Quả Theo 4 Paths

### Path 1: Happy Path - Khi Đúng (Nạp 50k Điện Thoại)

**Kết quả:**
- Hệ thống nhận diện **đúng intent** (Nạp tiền/Mua data)
- Thành công trong việc trigger Rich UI Component (Native Widget của MoMo)

**Trải nghiệm người dùng:**
- Không chỉ là text, Moni gọi thành công Rich UI Component
- Tự động trích xuất **ngữ cảnh cá nhân** (số điện thoại của user)
- Đưa ra các **gói cước cụ thể** (VT1, V50K)
- Cung cấp các **nút Action (Quick Replies)** để chốt sale ngay lập tức

**Đánh giá:**
- ✅ Trải nghiệm **O2O (Online to Offline action) xuất sắc**
- ✅ Rút ngắn funnel từ "hỏi" sang "hành động" (1-click conversion)
- ✅ Context awareness tốt

---

### Path 2: Ambiguous Path - Khi Không Chắc ("Chuyển Tiền Giúp Mình")

**Kết quả:**
- **Thất bại trong Slot-filling** (hỏi bù thông tin)

**Hệ thống xử lý như thế nào:**
- ❌ Thay vì hỏi "Bạn muốn chuyển cho ai và bao nhiêu tiền?", hệ thống rơi vào luồng **Fallback ngớ ngẩn**
- ❌ Trả lời template chối bỏ: "Mình chỉ hỗ trợ thông tin tài chính..."
- ❌ AI không map được cụm từ "chuyển tiền" với bất kỳ tool hay intent nào

**Vấn đề:**
- Thiếu **Fallback Slot-filling State**
- Không có cơ chế hỏi lại thông tin cần thiết
- Trải nghiệm user bị hỏng

---

### Path 3: Correction/Complex Path - Khi Sai/Cần Sửa

Trải nghiệm ở path này chia thành **2 cực đối lập**:

#### Cực Tốt: Context Retention (Thống Kê Chi Tiêu)
- **Hiện tượng:** Khi user gõ "À nhầm, tháng trước", AI nhớ được context "thống kê chi tiêu" từ câu trước
- **Kết quả:** Update lại tham số thời gian **xuất sắc**
- ✅ **Coreference Resolution** rất ấn tượng
- ✅ Giữ được continuity trong conversation

#### Cực Tệ: Capability Hallucination (Lên Lịch/Ghi Chú)
- **Mâu thuẫn logic nặng nề:**
  - Câu trước: Moni chủ động "gạ" user: *"Bạn muốn Moni ghi chú vào thứ Sáu (10/04) không?"*
  - User chốt: *"Ghi chú 5 triệu nhé"*
  - Backend từ chối (API không cho ghi chú ở tương lai)
  - Moni quay xe: *"Moni không thể ghi chú cho ngày tương lai"*

**Vấn đề:**
- ❌ **Agentic Hallucination** - AI hứa hẹn việc mà backend không thể thực hiện
- ❌ System Prompt không dặn AI về ràng buộc của API (DateTime <= Now)
- ❌ Gây **ức chế người dùng** và mất niềm tin

---

### Path 4: Trust/Failure Path - Khi Mất Tin ("Tự Dưng Mất Tiền")

**Kết quả:**
- ✅ **Safe Fallback** được kích hoạt đúng cách

**Hệ thống xử lý:**
- Khi gặp keyword nhạy cảm ("mất tiền", "tự động chuyển"), kích hoạt **Guardrails**
- Vạch rõ ranh giới quyền hạn: *"Chỉ hỗ trợ ghi chú... hoàn toàn không can thiệp giao dịch"*
- Hướng dẫn user check lịch sử và báo CSKH

**Đánh giá:**
- ✅ Xử lý chuẩn mực về **Risk Management**
- ✅ Bảo vệ người dùng tốt
- ✅ Giữ được niềm tin

---

## 2. Điểm Hay Nhất (The Best Point)

### Sự Kết Hợp Mượt Mà Giữa NLU và Native UI

**Câu chuyện:**
Moni không cố gắng giải quyết mọi thứ bằng text (vốn rất khó nhìn với dữ liệu tài chính).

**Cách làm:**
- Khi có thể, nó gọi các **Component UI của MoMo** lên trực tiếp khung chat
  - Thẻ nạp Data
  - Danh sách thống kê (Bullet point)
  - Widget Nhắc nhở
  - Quick Reply buttons

**Kết quả:**
- ✅ **Rút ngắn funnel** từ "hỏi" sang "hành động"
- ✅ **Tăng conversion rate** đáng kể
- ✅ **Context awareness** vượt trội (Coreference Resolution)
- ✅ Trải nghiệm user như native app, không phải chat bot

---

## 3. Path Yếu Nhất (The Weakest Path)

### Luồng Xử Lý Tác Vụ Có Điều Kiện Ràng Buộc (Lên Lịch/Ghi Chú Tương Lai)

**Tên lỗi:** **Agentic Hallucination** (Ảo giác năng lực của Agent)

**Nguyên nhân gốc rễ:**
- System Prompt dặn AI: *"Hãy gợi ý giúp user ghi chú"*
- ❌ **Quên dặn AI:** *"Tham số DateTime của Tool ghi chú chỉ nhận giá trị <= Now"*
- Hậu quả: AI hứa hẹn việc mà Backend **không cho phép thực hiện**

**Tác hại:**
- ❌ **Ức chế tột độ** cho người dùng
- ❌ Mất niềm tin vào AI
- ❌ Gây **confusion** trong luồng tương tác

---

## 4. Sơ Đồ As-Is và To-Be

### Hiện Trạng (AS-IS Flow)

```
User: 
  "Lên lịch thứ 6 chuyển 5tr tiền nhà"
    ↓
Moni (LLM - Analysis):
  ✓ Biết không có Tool "Chuyển tiền"
  ✓ Biết có Tool "Ghi chú"
    ↓
Moni (Reply - 🔴 LỖI SINH RA ĐÂY):
  "Không chuyển được. Ghi chú vào thứ 6 (10/04) nhé? Cho xin số tiền."
  ❌ Tự hứa việc không làm được
    ↓
User:
  "Ghi chú 5 triệu"
    ↓
Moni (Tool call):
  SaveExpense(amount=5000000, date=2026-04-10)
    ↓
Backend (API Response):
  ❌ Error: "Date cannot be in the future"
    ↓
Moni (Post-Error Reply):
  "Không ghi chú tương lai được. Ghi cho hôm nay nhé?"
    ↓
User:
  😠 Khó chịu, mất niềm tin
```

### Giải Pháp Kỳ Vọng (TO-BE Flow)

```
User:
  "Lên lịch thứ 6 chuyển 5tr tiền nhà"
    ↓
Moni (LLM + Logic Router - Analysis):
  Intent: Lên lịch chuyển tiền (Tương lai)
  Ràng buộc hệ thống:
    ✗ Không thể execute chuyển tiền
    ✗ Không thể ghi log chi tiêu ở tương lai
  Cơ hội Cross-sell: MoMo có tính năng "Nhắc nhở thanh toán/Lên lịch"
    ↓
Moni (Intelligent Reply):
  "Moni không thể tự động chuyển tiền giúp bạn. Nhưng Moni đã tạo sẵn 
   một thẻ Nhắc nhở chuyển tiền vào Thứ 6 này. Bạn kiểm tra thông tin 
   và bấm 'Lưu' nhé!"
    ↓
UI Render:
  [Thẻ Nhắc Nhở Widget]
    📅 Ngày: 10/04/2026 (Thứ 6)
    💰 Số tiền: 5,000,000đ
    [🔘 Lưu]  [✕ Hủy]
    ↓
User:
  Bấm [Lưu] (Chạm 1 nút) ✅
    ↓
✅ Xong việc - User hài lòng
```

---

## 5. Đề Xuất Cải Thiện

### BỚT (Remove) - Xóa Bỏ

| # | Hành động | Chi tiết | Lý do |
|---|-----------|---------|-------|
| 1 | Xóa các **proactive prompts** dẫn dụ user vào ngõ cụt | Xóa các gợi ý chủ động chỉ bề ngoài hỗ trợ | Tránh **capability hallucination** |
| 2 | Cấm Agent gợi ý ghi chú tương lai | Set **strict rule trong System Prompt** | Giữ agreement với backend capabilities |
| 3 | Xóa Fallback template chối bỏ | Hỏi lại / render UI thay vì nói "không hỗ trợ" | Tăng conversion, tránh dead-end |

---

### THÊM (Add) - Bổ Sung

| # | Hành động | Chi tiết | Kỳ vọng |
|---|-----------|---------|--------|
| 1 | **Intent Mapping Layer** | Map Intent user → Native Features của MoMo | Chuyển hướng request tương lai sang "Reminders Tool" thay vì "Expense Log Tool" |
| 2 | **Fallback Slot-filling State** | Khi user gõ "chuyển tiền", render danh bạ hoặc hỏi "Chuyển tới Ví/Ngân hàng nào?" | Tránh dead-end reply, tăng completion rate |
| 3 | **Constraint Validation Layer** | Với mỗi Tool, define rõ ràng điều kiện đầu vào | Tránh backend rejection errors |
| 4 | **Cross-sell Widget Factory** | Khi phát hiện intent nằm ngoài quyền hạn → tạo widget alternative | Chuyển thất bại thành cơ hội |

---

### ĐỔI (Change) - Thay Đổi

| # | Hiện trạng | Cần thay đổi thành | Lợi ích |
|---|-----------|------------------|--------|
| **1** | Logic xử lý Ambiguous mơ hồ (case "Chuyển tiền giúp mình") | Phải có **mandatory Slot-filling State**: render danh bạ gần đây hoặc hỏi lại | ✅ Hoàn tất request thay vì chối bỏ |
| **2** | System Prompt chỉ dặn về capabilities | **Bổ sung Constraint Definitions** cho mỗi Tool (DateTime range, Account limits, etc.) | ✅ Giảm hallucination, tăng trust |
| **3** | Error handling sau backend rejection | **Proactive suggestion flow**: không chỉ báo lỗi, mà gợi alternative (Reminder thay vì ExpenseLog) | ✅ Recover từ lỗi, tăng user satisfaction |

---

## 6. Tóm Tắt & Hành Động Tiếp Theo

### Điểm Mạnh Cần Giữ
- ✅ Integration NLU + Native UI
- ✅ Context awareness & Coreference resolution
- ✅ Safe fallback cho security-critical operations

### Điểm Yếu Cần Khắc Phục
- ❌ Agentic hallucination (gợi ý tương lai)
- ❌ Thiếu intent mapping (chuyển đổi sang alternative native tools)
- ❌ Fallback reply template toàn nói "không hỗ trợ"

### Ưu Tiên Cao
1. **Thêm Constraint Validation Layer** - Tránh backend errors
2. **Thêm Intent Mapping** - Chuyển hướng tương lai → Reminders Tool
3. **Thay đổi Error Handling** - Gợi alternative thay vì từ chối

---

*Báo cáo này được biên soạn nhằm cải thiện trải nghiệm người dùng với Moni và tăng độ tin cậy của hệ thống.*
