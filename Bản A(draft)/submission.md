# Day 17 Submission
---

**Student:** Lê Văn Quang Trung-2A202600383

**Date:** 2026-04-24

**Product idea:** AI TA chatbot cung cấp course-specific answers 24/7 cho first-year CS students tại VinUniversity, tuân thủ PDPA.

---

## 1. MVP Boundary Sheet

**Riskiest Assumption:**
> Sinh viên sẽ tin tưởng và sử dụng chatbot thay vì ChatGPT/Google/TA, với adoption rate ≥80% trong first-year class.

**In-Scope** (tính năng cốt lõi bắt buộc để test giả thuyết):

- [ ] **Course-grounded RAG Q&A** — Hệ thống retrieval từ course materials (slides, labs, assignments) để trả lời câu hỏi programming với citations
  - Test giả định: Sinh viên sẽ dùng nếu câu trả lời đúng với khóa học và có nguồn rõ ràng
- [ ] **Chat interface tích hợp vào LMS** — Giao diện chat đơn giản trong Canvas/Moodle, hỗ trợ text input và hiển thị citations
  - Test giả định: Access path rõ ràng (LMS integration) sẽ tăng adoption
- [ ] **Response time ≤5 phút** — SLA cho mỗi câu hỏi (retrieval + generation)
  - Test giả định: Instant response là yếu tố quan trọng với Need #1 (immediate help)

**Out-of-Scope** (tính năng tốt nhưng không cần cho MVP):

- [ ] **ReAct agent với tool calling** — Chỉ dùng simple RAG retrieval, không cần multi-step reasoning cho MVP
- [ ] **Hybrid deployment** — Bắt đầu với cloud deployment (vi phạm PDPA), có thể chuyển on-premise sau khi validate
- [ ] **TA dashboard & analytics** — Chỉ focus vào student experience, không build admin tools trong MVP
- [ ] **Multi-course support** — Chỉ support 1 môn (Intro to Programming) trong pilot
- [ ] **Mobile app** — Chỉ web/LMS integration
- [ ] **Voice input/output** — Text only

**Non-Goals** (ranh giới đỏ — sản phẩm sẽ KHÔNG làm):

- [ ] **Hỗ trợ môn học năm 2, 3, 4** — Chỉ first-year courses
- [ ] **Hỗ trợ môn không phải CS** (Math, Physics) — Chỉ programming labs
- [ ] **Fine-tune model riêng** — Dùng off-the-shelf LLM với RAG
- [ ] **Tự động update course materials** — TA/lecturer phải upload manually
- [ ] **Xử lý PII data** — Trong MVP, không collect student PII (anonymous queries)
- [ ] **Revenue generation** — Đây là internal tool, không có pricing model

---

**Checkpoint 1 — Kill questions:**

1. Nếu cắt thêm 1 tính năng trong In-Scope, khách hàng còn nhận được giá trị cốt lõi không?
   - Cắt RAG Q&A → không còn value
   - Cắt LMS integration → adoption rate giảm
   - Cắt 5-min response → không giải quyết Need #1
   → Tất cả đều cần thiết

2. In-Scope có 3 tính năng (không quá 5)

3. Non-Goals có nhiều thứ team muốn làm: hybrid deployment, ReAct agent, TA dashboard

---

## 2. PRD Skeleton

### Problem Statement
First-year CS students tại VinUniversity mất 2-4 giờ mỗi lần bị stuck với bug đơn giản vào tối/cuối tuần vì không có TA hỗ trợ, dẫn đến deadline anxiety, chất lượng bài nộp thấp, và drop-out rate tăng.

### Target User
First-year CS students (150-200 sv/khóa) tại VinUniversity, tuổi 18-20, thường làm bài tập vào 19h-23h và cuối tuần, có ít kinh nghiệm debug, cảm thấy panic khi gặp bug và không có TA để hỏi.

### User Stories

**Story 1:**
> As a first-year CS student, I want to ask a programming question at 10pm and get a course-specific answer with citations within 5 minutes, so that I can continue my lab without losing momentum or missing the deadline.

**Story 2:**
> As a first-year CS student, I want to see the source (slide/page) cited in the answer, so that I can verify the information against my course materials and trust it won't mislead me.

### AI-Specific

**Model Selection:**
- Model: GPT-4o-mini
- Lý do chọn:
  - Cost: ~$0.001-0.005/query → với 240 students × 3 queries/week = ~$45-225/tháng → chấp nhận được
  - Accuracy: State-of-the-art cho programming questions, instruction-following tốt
  - Latency: ~5s API call → tổng response time ≤5 phút (bao gồm retrieval)
  - Context window: 128K → đủ cho nhiều slides + query + response
- Trade-offs chấp nhận được:
  - Cloud deployment (vi phạm PDPA) — chấp nhận trong MVP để test adoption nhanh
  - Không fine-tuned — dùng RAG là đủ cho domain specificity
- Trade-offs không chấp nhận được:
  - Latency >30s — sẽ làm student bỏ cuộc
  - Hallucination rate >10% — sẽ làm mất trust

**Data Requirements:**
- Nguồn: Course materials từ Intro to Programming (CS101) — slides PDF, lab specifications, assignment descriptions, past exam questions
- Owner: VinUniversity CS department — lecturer/TA upload materials vào knowledge base
- Update frequency: Mỗi học kỳ — upload materials cho khóa học mới
- Rủi ro về data quality:
  - Materials không có trong digital format → cần scan/OCR (có thể sai)
  - Materials outdated → retrieval trả lời sai
  - Mitigation: TA review trước khi upload, hit rate monitoring

**Fallback UX:**
- Chiến lược: **Human-in-the-loop** (vì AI có thể hallucinate và đưa ra code suggestion sai → ảnh hưởng đến learning outcome)
- Trigger:
  - Confidence score < 80% (từ model)
  - HOẶC response chứa pattern "I'm not sure", "I don't have enough information"
  - HOẶC retrieval hit rate < 50%
- Hành động:
  - Hiển thị: "I'm not confident about this answer. Let me connect you with a TA."
  - Tự động chuyển sang TA queue (email hoặc chat)
  - Thông báo estimated wait time: "A TA will respond within 2 hours during business hours"
- User options:
  - [Ask again with rephrased question] — thử lại với câu hỏi khác
  - [Browse course materials manually] — link đến slides/LMS
  - [Join waitlist for TA] — queue vị trí trong hàng đợi
- User có thể override AI: **Có** — nút "Ask a TA instead" luôn hiển thị

### Dependencies & Constraints

- API / Service cần tích hợp:
  - OpenAI API / Anthropic Claude API
  - FAISS / Pinecone hoặc vector DB khác
  - LMS integration (Canvas/Moodle API)
- Timeline constraint: Pilot 4 tuần trong học kỳ hiện tại
- Budget constraint: < $1,000/tháng cho API costs trong pilot
- Legal / Compliance constraint: Vi phạm PDPA với cloud deployment — cần approval từ VinUniversity legal team; lộ trình chuyển on-premise sau pilot

---

## 3. Hypothesis Table

### Hypothesis 1 — Course-grounded RAG Q&A

> "Chúng tôi tin rằng **course-grounded RAG Q&A** sẽ giúp **first-year CS students** đạt được **trustworthy answers với citations trong 5 phút**.
> Chúng tôi sẽ biết mình đúng khi thấy **RAG hit rate >70%** và **user satisfaction score >4.0/5.0** trong **4-week pilot**."

**Riskiest assumption phía sau hypothesis này:**
> Students sẽ trust câu trả lời từ chatbot nếu có citations từ course materials, với accuracy ≥70%.

**Cách test assumption này với chi phí thấp nhất:**
> Manual QA: Lấy 50 câu hỏi thực tế từ students, chạy qua system, đánh giá độ chính xác và chất lượng citations. Nếu >70% câu trả lời đúng và citations relevant → assumption plausible.

---

### Hypothesis 2 — LMS Integration

> "Chúng tôi tin rằng **integrate chatbot vào LMS** sẽ giúp **first-year CS students** đạt được **adoption rate ≥80%**.
> Chúng tôi sẽ biết mình đúng khi thấy **≥80% students trong pilot class sử dụng chatbot ít nhất 3 lần trong 4 tuần**."

**Riskiest assumption phía sau hypothesis này:**
> Students sẽ dùng chatbot nếu nó có sẵn trong LMS (không cần sign-up riêng).

**Cách test assumption này với chi phí thấp nhất:**
> Landing page test: Tìm 50 students từ class đó, demo chatbot, hỏi "Bạn có dùng nếu nó có trong Canvas không?" → nếu >80% nói có → assumption plausible.

---

### Hypothesis 3 — Response Time ≤5 phút

> "Chúng tôi tin rằng **response time ≤5 phút** sẽ giúp **first-year CS students** đạt được **immediate help**.
> Chúng tôi sẽ biết mình đúng khi thấy **trung bình response time <2 phút** và **<10% queries timeout** trong pilot."

**Riskiest assumption phía sau hypothesis này:**
> Students sẽ chấp nhận chatbot nếu response time <5 phút, nhưng sẽ bỏ nếu phải chờ >10 phút.

**Cách test assumption này với chi phí thấp nhất:**
> Load test: Simulate 50 concurrent users, đo latency của retrieval + generation. Nếu 95th percentile <2 phút → assumption plausible.

---

## 4. PMF Scorecard

**Aha Moment:**
> Student lần đầu tiên nhận được câu trả lời **course-specific** với citations rõ ràng, và nhận ra: "Ồ, cái này đúng với cách giáo viên giảng dạy — nó giúp tôi hiểu bài thay vì chỉ copy code."

Hành vi cụ thể phản ánh Aha Moment:
> Student sau khi nhận câu trả lời, **click vào citation link** để xem source, và **đánh giá answer là "Helpful"** (thumbs up), và **quay lại dùng chatbot lần 2 trong vòng 24h**.

**Actionable Metric:**
> **"Aha Moment Rate"** = % của users who, trong first 3 sessions:
> 1. Nhận được câu trả lời có citations
> 2. Click vào ít nhất 1 citation link
> 3. Đánh giá answer là "Helpful" (thumbs up)
> 
> Công thức: (Sessions meeting all 3 criteria) / (Total first 3 sessions per user)

**PMF Method:**
> [x] **Sean Ellis Test** — ngưỡng: >40% "Very disappointed"
> 
> Cách đo: Sau 4-week pilot, survey users: "How would you feel if you could no longer use the TA chatbot?"
> - Very disappointed
> - Somewhat disappointed
> - Not disappointed
> - N/A (I no longer use it)
> 
> Ngưỡng PMF: >40% chọn "Very disappointed"
> 
> **Retention Curve** — ngưỡng: D7 > 30% (vì đây là educational tool, student dùng theo học kỳ)
> 
> **Aha Moment tracking** — ngưỡng: 60% users đạt Aha Moment trong first 3 sessions

**Vanity Metrics tôi sẽ KHÔNG dùng:**
- [ ] Total number of queries (có thể do 1 user spam)
- [ ] Total sign-ups (không phản ánh engagement)
- [ ] Pageviews của chat interface (không biết user có được value không)
- [ ] Average response time (chỉ đo hệ thống, không đo user satisfaction)

## 5. Self-assessment

**Mắt xích yếu nhất:**

> **Hypothesis & PMF** — Tôi còn yếu trong việc xác định **Aha Moment** thực sự và **metric actionable**. Tôi nghi ngờ "click citation" có phải là Aha Moment thực sự không, hay chỉ là indicator của curiosity. Cũng chưa chắc Sean Ellis Test có áp dụng được cho educational internal tool không (students có thể "Very disappointed" vì mất quyền truy cập chứ không phải vì product value?).

**Open questions cần giải đáp tiếp:**

1. Đối với internal tool (không phải commercial SaaS), PMF criteria có khác không? D30 retention có meaning không nếu học kỳ chỉ 12 tuần?
2. Aha Moment cho educational product: Có phải là "lần đầu hiểu concept nhờ chatbot" hay là "lần đầu trust đủ để delegate cả lab"? 
3. Hypothesis về RAG hit rate >70%: Hit rate là đủ, hay cần precision@5? Có metrics nào tốt hơn cho educational RAG?

---
