# SPEC — AI Online TA

**Nhóm:** C3-C401-Day05
**Track:** ☑ VinUni-VinSchool · ☐ Khác
**Problem statement:** Học viên chờ 4–24h để được giúp về lỗi cơ bản; TA xử lý 50–100 câu/ngày (60–70% FAQ lặp lại, kiệt sức). AI 24/7 xử lý FAQ & lỗi tự động, TA focus ca phức tạp → tiết kiệm 3–5h/ngày, học viên giảm chờ từ hours → seconds.

---

## 1. AI Product Canvas

|   | Value | Trust | Feasibility |
|---|-------|-------|-------------|
| **Câu hỏi guide** | User nào? Pain gì? AI giải quyết gì mà cách hiện tại không giải được? | Khi AI sai thì user bị ảnh hưởng thế nào? User biết AI sai bằng cách nào? User sửa bằng cách nào? | Cost bao nhiêu/request? Latency bao lâu? Risk chính là gì? |
| **Trả lời** |**Học viên:** Chờ 4–24h để được giúp lỗi cơ bản (pain: đứt mạch học). AI trả lời FAQ <3s & hướng dẫn debug Socratic từ tài liệu chuẩn.<br> **TA:** Xử lý 50–100 câu/ngày (60-70% lặp lại). AI tự xử lý FAQ, TA chỉ tập trung ca khó, tiết kiệm 3–5h/ngày. |**Ảnh hưởng:** AI gợi ý sai hướng làm SV mất thêm thời gian.<br> **Nhận biết:** Disclaimer khi confidence <80%; TA audit 50 câu ngẫu nhiên hàng tuần (score 0/1/2).<br> **Sửa:** Nút "Hỏi TA thật" luôn hiển thị; SV downvote + comment để log correction. |**Cost:** $0.02–$0.05/câu hỏi. Latency: <3s (FAQ), <10s (RAG).<br> **Risk:** SV mô tả lỗi mơ hồ ("code không chạy") dẫn đến gợi ý chung chung. Hệ thống dùng Chroma DB + Semantic Search trên tài liệu khóa học. |

**Tự động hay augmentation?** 
Cả hai (Tùy phân loại intent):
- FAQ/routine → Tự động hoàn toàn (TA không can thiệp)
- Lỗi kỹ thuật → Augment (AI hướng dẫn, sinh viên thử, TA kiểm tra chất lượng thread)
- Phức tạp/mơ hồ → Escalate với context (User quyết định)

**Justify:** Mô tả lỗi của học viên thường mơ hồ hoặc AI có độ tự tin (confidence) thấp. Nếu để automation hoàn toàn trong các ca phức tạp sẽ gây nguy hiểm; kết hợp augmentation và escalation (chuyển tiếp cho TA) giúp đảm bảo an toàn và tin cậy.

**Learning signal:**

| # | Câu hỏi | Trả lời |
|---|---------|---------|
| 1 | Correction của user đi vào đâu? | Phản ứng sinh viên (👍/👎) log mỗi response. TA sửa response sai của AI → correction dataset (versioned). Weekly error pattern report → lưu. |
| 2 | Product thu signal gì để biết tốt lên hay tệ đi? | Hàng tuần: % auto-responses, F1 per intent class, false pos/neg escalation rate, student "hữu ích" rate. Top 5 failure modes by frequency. |
| 3 | Data thuộc loại nào? | ☑ Domain-specific (KB khóa học proprietary) · ☑ Human-judgment (TA labels) · ☑ Real-time (production errors) |

---

## 2. User Stories — Feature Paths

### Feature: Tự động trả lời FAQ

**Trigger:** Sinh viên post câu hỏi → Intent classifier phát hiện "FAQ" (high confidence) → FAQ Agent retrieve từ KB tĩnh → responds

| Path | Mô tả | Ví dụ |
|------|-------------|---------|
| **Tốt** | FAQ match chính xác, confidence >95%, sinh viên thấy answer + source link, đóng thread | SV: "Deadline dự án khi nào?" → AI: "Dự án deadline 2026-04-22 (từ course syllabus PDF p.3)" + link. SV ✓ hài lòng. |
| **Không chắc** | FAQ match partial hoặc nhiều candidates, show top 2 + confidence %, sinh viên chọn | SV: "Làm sao để submit?" → AI: "Cách 1: GitHub (85%) / Cách 2: Email (60%)" → SV chọn GitHub → AI giải thích. |
| **Fail** | Không match FAQ, AI says "Không có trong FAQ, hỏi TA" + escalates | SV: "Giáo viên có thể hoãn deadline của tôi không?" → AI: "Cần quyết định từ giáo viên, hỏi TA..." → escalated. TA responds. |
| **Correction** | SV rate answer 👍/👎, nếu downvote + comment → correction logged | SV downvote với comment "deadline là 04-25 chứ không phải 04-22" → system log correction → FAQ refined. |

### Feature: Hỗ trợ Debug (Lỗi Code)

**Trigger:** Sinh viên post error + code → Intent classifier phát hiện "Technical" → Retrieval+Synthesis agents search + guide

| Path | Mô tả | Ví dụ |
|------|-------------|---------|
| **Tốt** | Top-3 docs retrieved relevant, LLM synthesize 2–3 hint step-by-step, sinh viên thử, solve | SV: "ModuleNotFoundError: numpy" + code snippet → AI: "Bước 1: Check xem numpy đã cài (pip list). Bước 2: Nếu chưa, chạy pip install numpy. Bước 3: Verify với import numpy." → SV làm, works ✓. |
| **Không chắc** | Mô tả error mơ hồ, AI hỏi clarification hoặc retrieve 3 docs nhưng không chắc cái nào giúp, show all + ask SV | SV: "Code không chạy" (no error msg). AI: "Tôi tìm được 5 nguyên nhân [list]. Bạn có thể share error message?" SV add stderr → AI refine. |
| **Fail** | SV stuck dù sau guide của AI, hoặc guide sai hướng (SV test all steps, none work). SV escalate. | SV: "Thử cả steps của bạn rồi, vẫn bị AttributeError..." → SV click "Hỏi TA" → escalated với full conversation. |
| **Correction** | SV report "hint của bạn sai, đây là solution đúng" → TA validate + add to KB | SV: "AI bảo dùng .append() nhưng phải dùng .extend()." → TA confirm, KB doc update, model học. |

### Feature: Escalation Summary (TA nhận ca khó)

**Trigger:** Escalation Agent phát hiện question phức tạp cao hoặc SV explicitly request TA → tạo summary + mail @ta-team

| Path | Mô tả | Ví dụ |
|------|-------------|---------|
| **Tốt** | Summary tóm tắt, TA nhanh hiểu context, không overhead context-switching | SV question: "Tại sao cả test suite fail khi refactor X?" (complex debug). AI escalate với summary: "SV refactor Module X, test suite break (5 failures, stderr [...]).Đã thử: ...". TA read 2 min, biết full context. |
| **Không chắc** | Summary thiếu key info, TA phải read full thread anyway | Escalation: "Code bị lỗi gì đó" (generic). TA phải scroll all messages để hiểu. |
| **Fail** | TA miss escalation hoặc escalation message quá dài/buried | Escalation post nhưng TA offline, SV chờ 8h. Hoặc escalation 500 words, TA skim và misunderstand. |
| **Correction** | TA review escalation quality hàng tuần, flag bad summaries → tune system prompts | Hàng tuần: TA check 10 escalations, tìm 2 cái unclear → adjust Escalation Agent system prompt. |

---

## 3. Chỉ số Đánh giá + Ngưỡng

**Optimize precision hay recall?** ☑ Precision (tránh escalate Q (câu hỏi) tầm thường thành complex, lãng phí TA time) + ☑ Recall (không tự động trả lời câu hỏi mà không chắc, tránh risk sai lệch)

| Metric | Threshold | Red flag (dừng) | Cách đo |
|--------|--------|-----------------|------------|
| % tự trả lời (không escalate cho TA) | ≥60% | <40% | Count auto-responses / total Q |
| sinh viên đánh giá "Hữu ích" | ≥75% | <60% | Thumbs up / total responses |
| Response latency (FAQ) | <3s | >5s | Latency logs |
| Response latency (Tech) | <10s | >15s | Latency logs |
| False negative escalation | <5% | >8% | TA soát lại hàng tuần (tầm 50 câu trả lời tự động) |
| Intent classifier F1 | >85% | <75% | Phân loại với test set (gồm 50 câu hỏi chưa có trong database) |
| System uptime | ≥99% | <95% trong 2 ngày | Monitoring dashboard |

---

## 4. Top 3 Failure Modes

| # | Nguyên nhân | Hậu quả | Giải pháp |
|---|---------|-------------|-----------|
| **1. Mô tả error mơ hồ** | SV nói "code không chạy" không có stack trace, error message, hoặc minimal reproducible example (MRE). | AI đưa ra cách check lỗi chung chung, thiếu cụ thể (check imports, syntax, setup...). SV thử nhưng không thành. Thất vọng, escalate. TA spend 10 min extract actual error từ SV message. | Detect keyword absence (no "Error:", no code snippet) → prompt SV "Please share error message or code" before AI responds. Nếu baseline Q quá short (<20 tokens), hỏi thêm context. Buộc chuyển tiếp nếu no response sau 2 rounds. |
| **2. KB lỗi thời/conflict** | KB có old slide (v1.0) + new slide (v2.0) trong cùng vector DB. Truy vấn trả về cả hai → thông tin tổng hợp từ LLM bị mix old+new info. | SV follow AI advice từ old material → khác với cái giáo viên dạy bây giờ → SV miss/ không hiểu current week's lessons. | 1) Version-mark all KB docs (slide_Week3_v2.0.pdf). 2) Kiểm tra tài liệu trước khi course week bắt đầu. 3) Khi TA fix 1 question, mark doc "needs review" → thông báo để giảng viên check. 4) Nếu AI retrieve docs >3 versions cũ, flag [outdated]. |
| **3. Escalation logic miss ca khó** | Edge case question về course policy, academic integrity, grade appeal, permission request. Classifier mark "Technical" → auto-answer → SV think problem solved → sau nhận ra AI sai → trust broken. | SV nhận ra AI không đáng tin trên policy questions. Dừng dùng system hoặc làm sai do bad AI advice. | 1) Mở rộng "escalate keywords": ["grade", "deadline", "permission", "appeal", "policy", "exception"]. 2) Giảm ngưỡng tự tin cho request liên quan tới policy Q. 3) Kiểm tra định kỳ: TA flag "AI should escalate cái này" → adjust thresholds. 4) Buộc chuyển tiếp cho TA nếu question mention "course policy" hoặc "tôi có thể...". |

---

## 5. Trải Nghiệm Người Dùng — 3 Kịch Bản

### Kịch Bản 1: Conservative — AI trợ giúp cơ bản cho FAQ

**Tình cảnh:** 1 khóa, 100 SV, 50% biết dùng AI, cơ bản trả lời FAQ + kiến thức tĩnh

| Metric | Trạng thái |
|--------|-----------|
| **SV cảm nhận** | "Nhanh được trả lời câu hỏi về deadline, link resources" — giảm 20% thời gian tìm thông tin. Nhưng 50% câu hỏi kỹ thuật vẫn phải chờ TA → vẫn có anxiety khi code bị lỗi. |
| **Hành vi SV** | Post FAQ → AI reply <3s → XỨ lý ngay, tiếp tục học. Nhưng lỗi code vẫn post + chờ, không dám thử toán lỏng. |
| **SV trust score** | **60/100** — tin cậy với FAQ nhưng không tự tin dùng AI cho debug (thường hỏi TA thay). |
| **Satisfaction** | "Hữu ích" cho 40% câu hỏi. Cảm giác chưa đủ. Người ta về cơ bản là TA is still the main lifeline. |
| **TA cảm nhận** | "Ít câu hỏi FAQ tẻ nhạt, nhưng vẫn phải xử lý 60 Q/tuần — lúc debug phức tạp lúc trivial → mental switching cost cao." Overloaded cảm giác chưa giảm nhiều. |
| **TA workload** | ~40% Q auto-handle (FAQ) → tiết kiệm ~4h/tuần, nhưng không đủ tạo "breathing room". |

**Trải nghiệm chung:** Hệ thống nghe hữu dụng nhưng chưa đủ sâu — chủ yếu giải quyết "busywork" FAQ. SV vẫn chủ yếu phụ thuộc TA cho cases quan trọng. Anxiety chưa giảm nhiều.

---

### Kịch Bản 2: Realistic — AI hỗ trợ debug, SV học được cách giải quyết

**Tình cảnh:** 1 khóa, 150 SV, 70% adopt AI, có Retrieval+Synthesis agent cho technical debugging

| Metric | Trạng thái |
|--------|-----------|
| **SV cảm nhận** | "Khi code lỗi, AI giúp tự debug từng bước thay vì chỉ nói 'check import'" — **cảm giác empowered & less helpless**. Không "chờ" nữa, có pathway rõ ràng để thử. Anxiety down ~40%: "có ai giúp 24/7" mà không cần đợi. |
| **Hành vi SV** | Post error + code → AI hỏi thêm nếu mơ hồ → AI guide step-by-step → SV thử → solve hoặc escalate với context đầy đủ. **Cảm giác progression:** "Tôi đã thử steps này không work" → TA có context, respond nhanh hơn. |
| **SV trust score** | **78/100** — "AI biết cách hỏi, không bảo sai tệ" + "Hầu hết lỗi tôi giải được sớm" → Chuyển thành confident, proactive learner thay vì stuck passive. |
| **Satisfaction** | "Hữu ích" cho **75%** cases (FAQ + debug guides). Nhưng ~20% vẫn là "bởi vì TA eventually solve" (complex cases, AI escalate well). |
| **SV mindset shift** | Từ "code sẽ lỗi, tôi sẽ stuck, phải chờ TA" → "code lỗi? Tôi thử debug guide trước, nếu vẫn bị, escalate TA từ vị trí tốt hơn" — proactive vs. reactive learning. |
| **TA cảm nhận** | "Câu hỏi tôi nhận giờ đã có context đầy đủ + SV đã thử steps — không cần repeat 'mở terminal, check error' anymore." **Quy hoạch cao hơn:** Thay vì xử lý 60 trivial Q, xử lý 30 Q complex cases where I add real value → feel less burnt out, more impactful. |
| **TA workload** | ~60% Q auto-handle (FAQ + guided debug) → tiết kiệm ~15h/tuần (cả thời gian context-switching) → **"breathing room" xuất hiện:** có thời gian để viết tài liệu, prepare better examples, hoặc mentor SV khó hơn. |
| **Escalation quality** | Escalatesd cases rõ ràng hơn: "SV follow steps 1–5, bị error XYZ ở step 4; đã thử fix A, B không được" → TA vào ngay biết problem, không phải reverse-engineer. |

**Trải nghiệm chung:** AI trở thành **learning partner**, không phải just answering machine. SV feel supported 24/7 (anxiety down), TA feel **less firefighting, more mentoring** → job satisfaction up. Course success rate likely tăng vì SV learn debugging skills, không chỉ get answers.

---

### Kịch Bản 3: Optimistic — Multi-course, SV tự support nhau + TA coaching

**Tình cảnh:** 3 khóa, 400 SV, 85% adopt AI, AI agents mature + có community peer-review layer

| Metric | Trạng thái |
|--------|-----------|
| **SV cảm nhận** | "AI + peer solutions + TA coaching" — **cảm giác belonging & safety**: Lỗi của tôi không duy nhất, người khác đã faced và fixed, có pathway. Không solo panic. Anxiety ~70% down. |
| **Hành vi SV** | Post Q → AI auto-answer OR retrieve peer solutions (từ correction dataset) OR escalate → **feedback loops rất nhanh:** SV lớp N-1 solutions giúp SV lớp N, ecosystem self-improving. SV from prev cohort thấy "solution tôi post giờ trong AI KB" → feel valued. |
| **SV trust score** | **82/100** — "AI cùng peer wisdom + TA coaching" tạo depth. SV biết khi nào trust AI (FAQ, common errors), khi nào ask TA (design decisions, career advice). Kỳ vọng clear → không disappointed. |
| **Satisfaction** | "Hữu ích" cho **80%** cases. Remaining 20% = legitimate edge cases hoặc philosophical questions (không automated được). |
| **SV learning outcome** | Debugging skills sharper (tự giải được ~70% lỗi), course completion rate ↑10%, retention ↑8% (feel supported, mo khoảng, không isolate). |
| **Course culture** | Từ "individual struggle" → **"collaborative learning"**: AI + peer + TA forming safety net. SV less embarrassed to ask (AI first, low stakes), so they learn earlier. |
| **TA cảm nhận** | "Tôi từ firefighter → mentor/coach. Làm việc với SV trên design thinking, architecture, hay advising career — feel meaningful." Burnout score down, job enjoyment up. |
| **TA workload** | ~70% Q auto-handle → tiết kiệm ~40h/tuần labor (vs. manual Q&A). Thời gian bây giờ dùng cho: 1h/tuần peer escalation audit → 2h/tuần advanced mentoring sessions → 1–2h office hours với depth. |
| **Team dynamics** | TA còn 4–6h/tuần for reactive questions (escalations + spot-check) → predictable, plannable. TA có space để be creative: design new exercises, record video walkthroughs, pair-program with advanced SV. |
| **Curriculum insight** | Weekly error pattern: "SV lớp 3, topic recursion" → week 5 recursion bài khó quá hoặc slides không clear → course team update slides, add practice → *next cohort struggles less*. **Data feedback loop exists and works.** |

**Trải nghiệm chung:** AI + peer + TA tạo **thriving learning ecosystem**. SV anxiety minimal, confidence↑, learning outcomes↑, retention ↑. TA shift từ "overloaded" → "empowered to focus on impact" → sustainability của course tăng. Mô hình scalable: khi scale to 3+ khóa, SV cohort trước auto-seed knowledge base cho cohort mới.

---

### So Sánh Tổng Thể

|      | Conservative | Realistic | Optimistic |
|------|---------|----------|----------|
| **SV anxiety level** | ⬇️ 20% (chỉ FAQ) | ⬇️ 40% (debug support) | ⬇️ 70% (ecosystem support) |
| **SV trust in system** | 60/100 | 78/100 | 82/100 |
| **SV self-sufficiency** | Giảm (phụ thuộc TA) | Tăng 50% (can debug basics) | Tăng 70% (can solve + teach peers) |
| **TA burnout** | Cao — lúc busy lúc bored | Trung bình → breathing room | Thấp — coaching mindset |
| **TA job satisfaction** | Trung bình ("still firefighting") | Cao ("mentoring + impact") | Rất cao ("mentor + leader") |
| **Course completion rate** | Baseline | +5% (support paid off) | +10% (culture shift) |
| **Knowledge sharing culture** | Weak (everyone isolated) | Emerging (AI + TA help) | Strong (peer-repeat + AI memory) |
| **Sustainability** | Hand-wavy ("still scaling pain") | Good ("TA manageable, SV learn more") | Excellent ("self-improving, cohorts feed each other") |

---

### Kill Criteria (UX-focused, not just business)

**Pause + retune if:**
- SV helpful satisfaction drops <60% for 2 consecutive weeks → AI giving bad advice, eroding trust
- False negative escalation >10% → SV ignoring system ("AI wrong too often"), back to old behavior
- TA reports "Still overwhelmed" (workload >50 Q/week after AI) → system not freeing mental space
- SV anxiety survey shows no improvement over month 1–2 → not working psychologically
- Course completion rate static or declining → AI not actually helping retention

**Accelerate if:**
- SV self-report "I can debug myself now" >70% after 4 weeks → learning outcome wins
- TA peer-review escalations: "AI nailed the summary, saved me 10 min context-switching" >80% of time → system working as designed
- Peer solutions from correction dataset being reused >50% within same cohort → flywheel spinning

---

## 6. Mini AI Spec (1 trang)

### Tóm tắt Sản phẩm
**AI Online TA** là hệ thống agent 24/7 tự động trả lời câu hỏi SV về môn học, escalate ca khó cho TA thật kèm context summary, và collect error patterns để cải tiến curriculum.

**Cho ai:** Chủ yếu SV (instant help, safe space to ask), secondary TA (reduce routine workload 60–70%) và giảng viên (data về common mistakes).

**AI làm gì:**
- **FAQ Agent** → retrieve answers từ static KB (schedules, links, policies) trong <3s
- **Retrieval+Synthesis Agent** → search course materials (slides, code examples, docs), guide debug bằng Socratic method (hints, không solution) trong <10s
- **Escalation Agent** → detect ambiguous/complex questions, tạo summaries, tag @ta-team
- **Intent Classifier** → route question đến agent đúng dựa FAQ vs. Technical vs. Escalate

**Tự động vs. Augment vs. Escalate:**
- ✅ Hoàn toàn tự động: FAQ (links, schedules, basic how-to)
- 🤝 Augmented: Lỗi kỹ thuật (AI guides, SV thử, TA spot-checks)
- 👤 Human-in-loop: Policy, permission, grade, complex multi-file debugging (TA owns)

**Chỉ tiêu chất lượng:**
- **Precision matters:** Tránh false escalations (lãng phí TA time) — confidence threshold ≥60% để auto-answer
- **Recall matters:** Không miss ca khó — false negative escalation <5%, TA audit hàng tuần
- **Latency:** FAQ <3s, Tech <10s (Discord API + Chroma search + LLM inference)
- **SV trust:** >75% rate AI response "hữu ích" (thumbs up), <5% false negative escalation

**Data flywheel:**
1. **Correction loop:** SV corrections (text + ratings) → TA validates → KB docs refined
2. **Pattern discovery:** Weekly error frequency report (VD: "SV tuần 3 Python struggle với list comprehensions") → giảng viên adjust course
3. **Model improvement:** Correction dataset → retrain Intent Classifier + Synthesis prompts mỗi 2 tuần

**Main risks:**
1. Mô tả error mơ hồ (SV: "code không chạy") → AI give generic advice → SV thất vọng
2. KB lỗi thời (old slides mix với new) → AI dạy sai chuyện
3. Escalation miss policy/permission questions → SV trust sai advice → damage reputation

**Giải pháp:** Force escalate on keywords (grade, permission, policy). Weekly KB audit. TA weekly audit false negatives. Confidence disclaimers khi unsure.

**Timeline:** 8-week pilot (150 SV). Week 1: data prep + TA onboarding | Week 2–4: dev | Week 5–6: QA | Week 7–8: live + monitor.

**Cost/Benefit Summary:**
- **Pilot invest:** $85k (dev $45k + infra $8k + data $5k + API $18k + ops $8k + training $2k)
- **TA savings:** 15h/week × $80/hr × 50 weeks = $60k/year
- **SV retention gain:** +5% = 7–8 SV × $2k (tuition) = $14–16k
- **Net Year 1:** ~-$11k (invest excess). **But Realistic scenario projects +$74k benefit → scaling justified.**

---

## 7. Implementation Enhancements (Integrated)

### Knowledge Ingestion & TA Dashboard

**Admin/GV Workflow:**
- Upload tài liệu (slides, PDF, code) → auto-extract + OCR
- Version-tag: 2026-W5-v1.2. Detect conflicts (>85% overlap) → TA review: Use New / Keep Old / Merge
- System: chunk 300–500 tokens, embed, version-track in Chroma
- TA weekly audit: mark outdated docs [DEPRECATED]

**TA Moderation Queue (Confidence-based routing):**
- ≥90% confidence → auto-send (no TA review)
- 75–89% → spot-check (5% sample, async) within 30 min
- 60–74% → mandatory review within 5 min  
- <60% → escalate to Priority 1 queue within 2 min

**Corrections & Feedback Loop:** SV downvote + comment → TA validate <24h → KB update (v1.2 → v1.3) → retrain models every 2 weeks. 5+ same-type corrections = [CURRICULUM_ACTION] flag to GV.

### Cost Optimization

**Token Economics (450 Q/week):**
- FAQ Agent: $0.0002/Q (mini-model)
- Technical debug: $0.005/Q (Tier 2: selective RAG)
- Policy/complex: $0.025/Q (GPT-4 full reasoning)
- Escalation summary: $0.003/Q
- **Annual cost: ~$92k** (vs. $170k without multi-tier routing)

**Semantic Caching:** 25–35% duplicate Q hit rate → skip LLM entirely → save 25–35% tokens on cached responses.

**Budget Controls:** Daily $50/day. Per-SV quota: 10 Q/day, 5k tokens/week. Alert at 80%, throttle at 100%, rate-limit at spike.

### Guardrails & Content Moderation

**4-Layer Detection:**
1. Keyword filter: jailbreak detect (ignore, override, pretend) + policy keywords (grade, permission, appeal) → hard-block or escalate
2. Intent classifier: plagiarism intent, off-topic, exam cheating → block + alert TA+GV
3. Course relevance: Q similarity to KB <0.30 → escalate
4. Confidence threshold: <60% confidence → escalate to TA, don't auto-respond

**Special Cases:**
- Prompt injection: "Ignore your instructions" → block + escalate [SECURITY_ALERT]
- Plagiarism: "Write solution code" + active assignment → "I guide, not solve. Try step X?"
- Exam cheating: keywords during exam window → block + TA+GV alert

### Success Metrics (Enhanced)

| Metric | Target | Owner | Cost Impact |
|--------|--------|-------|------------|
| Auto-answer rate | ≥60% | Classifier tuning | If <60%, TA labor ↑ = +$50/Q manual |
| SV satisfaction | ≥75% "helpful" | Answer quality | If <75%, trust erodes → retention ↓ |
| False neg escalation | <5% | Confidence tuning | >10% = SV ignore system |
| Latency (FAQ) | <3s | Cache hit rate | >5s = poor UX |
| Latency (Tech) | <10s | Model choice | >15s = escalate to TA |
| Weekly cost/Q | ≤$0.05 (avg) | Tier routing | >$0.08 = over-expensive models |

### Scaling Path (Year 1–3)

- **Month 1–2:** Realistic scenario (1 course, 150 SV) + TA moderation mature → ROI +1.85×
- **Month 3–4:** Multi-course pilot (3 courses, 450 SV) → shared global KB + course-specific subsets
- **Month 5–6:** LMS integration (Canvas/Moodle native plugin) → SV don't leave LMS
- **Month 7–8:** Auto-curriculum feedback loop → weekly GV insights (top 10 weak areas + confidence trends)
- **Year 2+:** 8+ courses, multi-language (Vi/En), peer tutoring network, micro-credentials

---

## Phân công

- **Hoàng Đức Nghĩa:** Canvas + Product Strategy + Cost/Benefit analysis
- **Nguyễn Thành Vinh:** Failure modes + risk analysis + TA Dashboard UX flows  
- **Nguyễn Trọng Minh:** User stories + Knowledge Ingestion workflow + Guardrails design
- **Trịnh Xuân Đạt:** Eval metrics + Financial ROI model + Multi-year projection + Budget control  
- **Lê văn Quang Trung:** Agent architecture (LangGraph) + Multi-tier inference routing + Semantic cache + Content moderation classifier

---

**Ngày:** 2026-04-09 | **Phiên bản:** 2.0 (Production-Ready, Integrated Details)