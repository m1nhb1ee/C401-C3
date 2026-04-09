# Individual Reflection — Nguyễn Trọng Minh

## 1. Role
Người viết User Stories và design Knowledge Base workflow. Có kèm theo fix Guardrails logic để AI không bịa chuyện.

## 2. Mình làm cái gì cụ thể
- **User Stories**: Viết 3 feature chính với các "path" khác nhau — không chỉ happy path mà còn "Không chắc", "Sai", "Sửa lại".
  - FAQ auto-reply path: khi nào confident, khi nào hỏi thêm, khi nào escalate
  - Debug code: cái flow từ user gửi error → AI hỏi rõ → guide từng bước
  - Escalation: tóm tắt context cho TA coi
  - Mỗi path có example thực tế + learning signal để track xem có tốt không
  
- **Knowledge Base**: Design cách organize tài liệu cho AI use.
  - Cấu trúc: slides/, code_samples/, faq.md, course_info.json
  - Dùng FAISS vector DB để search
  - Mỗi tài liệu lưu metadata (từ đâu, version gì, độ liên quan bao nhiêu)
  
- **Guardrails**: 3 rules để AI không bịa chuyện
  - Rule 1: Phân loại đúng — Assignment ≠ Lab ≠ Project (khác deadline khác cách handling)
  - Rule 2: Ghi nguồn — mọi info phải từ KB, mà phải ghi rõ lấy từ đâu
  - Rule 3: Thông tin thiếu → escalate cho TA luôn, đừng guess

## 3. SPEC mạnh ở đâu, yếu ở đâu
- **Mạnh**: User Stories framework mình viết khá chi tiết, 4 path mỗi feature có example cụ thể. Team dễ hiểu user journey có gì xảy ra khi user sai sai hay AI đoán sai. Quan trọng là từ đầu mình thiết kế feedback loop ("correction" path), không phải bổ sung sau.

- **Yếu**: Knowledge Ingestion workflow vẫn sơ sơ. Mình chỉ nói là "có versioning" nhưng không chi tiết implement như nào. Cơ bản là:
  - Tài liệu cũ vs. tài liệu mới — phải handle sao? (chỉ ngồi brainstorm chứ không code)
  - FAISS index khi scale lên hàng trăm doc sẽ nặng không?
  - Metadata schema format chuẩn ra sao? Ai define tags?
  - Guardrails chủ yếu text rules, chưa có code logic thực tế

## 4. Mình giúp mấy bạn cái gì
- Bounce ý tưởng với Nghĩa: cách chuyển Canvas "Value" thành concrete user stories
- Giúp Vinh refine Failure Modes: từ "user mô tả error mơ hồ" → code detection logic (nếu message <20 tokens + không có "Error:", hỏi lại)
- Review code plan của Quang Trung (LangGraph implementation) để chắc 3 escalation triggers chạy đủ

## 5. Mình học được gì
- **User Stories không phải chỉ "user hỏi → AI trả lời → user vui"**: Hơi ngô này, D5 tôi vẫn nghĩ vậy. Nhưng khi viết "Không chắc" + "Sai" paths, mới hiểu là real-world thực tế 60% cases không happy. Phải design recovery & fallback từ đầu. Cái robustness = 1 - (failure cost × failure probability), không phải chỉ optimize happy path.

- **Data bottleneck > engineering bottleneck**: FAISS search nhanh, nhưng nếu KB thiếu tài liệu hay tài liệu lỗi thời thì AI trả lời vô nghĩa. Mình nhận ra: learning signal phải từ TA sửa KB, không phải chiều tune LLM. Tuy nhiên manual sửa KB nặng → cần design process: TA notice bug → flag doc → auto-update KB.

- **Guardrails = feature, không chỉ "safety"**: Khi viết Rule 1–3, mình nhận ra: "AI không bịa chuyện" (grounding + reject hallucination) = selling point cho TA. TA trust → dùng → adoption up.

## 6. Làm lại sẽ thế nào
- Viết User Stories sớm hơn từ D5 tối, iterate luôn với Vinh + Nghĩa → phát hiện dependency sớm. (Ví dụ: Failure Mode "user mô tả mơ hồ" ảnh hưởng tới story "Không chắc" → phải design hỏi lại logic, không phải sau)

- Prototype Knowledge Ingestion: D6 sáng test thực tế với 5–10 slide giảng của khóa học, chứ không chỉ viết chữ. Khi đó sẽ phát hiện: versioning + metadata format cần concrete example, chứ không phải abstract.

- Collaborate sớm với Quang Trung & Vinh: "Mô tả error mơ hồ" → "detect token <20 + no Error keyword" = implementation detail, phải trong Agent code, không chỉ SPEC rule.

## 7. AI giúp gì / AI sai gì
- **Giúp**: Claude tốt ở brainstorm bad scenarios — gợi ý được "SV stuck sau khi follow guide" hay "TA miss escalation vì offline", những cái tôi không nghĩ tới. ChatGPT giúp structure user stories template nhanh.

- **Sai/hallucination**: Claude viết long context (tôi paste 3 pages spec vào), rồi nó gợi ý "auto-merge conflicts trong knowledge versioning" — nghe hợp lý nhưng thực tế knowledge versioning cần TA decision, không auto được. Suýt mình implement thêm merging logic không cần. Issue này là: Claude khi nhận long context quá nhiều, nó hallucinate features mà không biết thực tế constraints. Mình phải cut context, hỏi narrow hơn.

- **Claude hallucination cụ thể**: Một lần mình paste full Failure Modes + User Stories (~2000 tokens), Claude response nó confident về "auto-versioning strategy" nhưng sau khi implement thấy là không needed vì TA chỉ publish doc once per week. Bài học: khi context quá dài, AI "nhìn toàn bộ puzzle rồi tự assume missing piece" — cần validate lại, đừng trust 100%.

- **Thiếu**: AI không gợi ý được pattern end-to-end "User Story → Learning Signal → Metrics → Guardrails". Tôi phải map manually: mỗi failure path → 1 metric để track → 1 guardrail để prevent.