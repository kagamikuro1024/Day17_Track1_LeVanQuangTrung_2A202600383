# CHANGES SUMMARY — Bản A → Bản B

**Nguyên tắc:** Bản B là bản **sản phẩm thực tế** (trenches), không phải bài tập học thuật (academic exercise).

---

## OVERVIEW

| Aspect | Bản A (Draft) | Bản B (Revised) | Why changed |
|--------|---------------|-----------------|-------------|
| **Mindset** | Academic lab assignment | Product ready for trenches | AI feedback: "reads like an academic exercise" |
| **MVP Focus** | 3 In-Scope features, some nice-to-have | 1-2 core features only | MVP là test, không phải build đầy đủ |
| **Testing Method** | Hypothetical surveys ("Bạn có dùng không?") | Real behavior tracking (clicks, usage) | Mom Test failure — stated intent ≠ actual behavior |
| **Fallback UX** | Confidence score (không khả thi), hứa hẹn TA | Retrieval-based trigger, NO TA promise | LLMs không self-aware; TA không 24/7 |
| **PMF Metrics** | Vanity (click citation, thumbs up) | Actionable (task resumption, retention) | Click ≠ value; stressed users không rate |

---

## DETAILED CHANGES

### 1. MVP BOUNDARY SHEET

#### In-Scope Reduction

| Feature | Bản A | Bản B | Reason |
|---------|-------|-------|--------|
| Course-grounded RAG Q&A | ✅ In-Scope | ✅ In-Scope | **Killer feature** — giữ nguyên |
| LMS Integration | ✅ In-Scope | ❌ Out-of-Scope | Massive blocker (IT approvals, security reviews). Không cần để test core value. Standalone web app là đủ. |
| Response time ≤5 phút | ✅ In-Scope | ❌ Out-of-Scope | Đây là SLA/non-functional requirement, không phải feature bạn "build". Với GPT-4o sẽ tự nhiên <10s. |
| Standalone web app | Not mentioned | ✅ In-Scope (or implied) | Low-friction access, tránh bureaucratic hell. Có thể đưa vào Out-of-Scope nếu muốn tighter. |

**Kết luận:** In-Scope nên chỉ còn **1 feature** là RAG Q&A. Các thứ khác là implementation details.

#### Fallback Promise — CRITICAL CHANGE

**Bản A:**
- "A TA will respond within 2 hours during business hours"

**Bản B:**
- **KHÔNG HỨA HẸN TA RESPONSE TIME**
- Fallback chỉ cung cấp self-service: "Browse course materials", "Ask again", "Contact TA" (email form, no SLA)

**Why:** Students stuck at 10 PM Saturday — nếu bạn nói "TA responds within 2 hours" nhưng business hours là Monday 8 AM → bạn đã tạo false expectation và recreating chính problem bạn set out to solve.

---

### 2. PRD SKELETON

#### User Stories — From Feature to Behavior

| Bản A | Bản B | Problem |
|------|-------|---------|
| "I want to see the source (slide/page) cited" | "I want to see exactly which slide/page supports the answer, so that I can cross-check... and avoid learning incorrect concepts" | A: Mô tả UI ("see citations")<br>B: Mô tả hành vi và outcome ("cross-check", "avoid incorrect concepts") |
| "within 5 minutes" | "within 60 seconds" | A: SLA<br>B: Outcome-focused (fast enough để continue work) |

**Rule:** User Story phải mô tả **hành vi trong sản phẩm**, không mô tả **UI elements**.

#### Model Selection — Trade-offs Rõ Ràng

**Bản A:** Nói chung chung về cost, latency, accuracy.

**Bản B:**
- **Trade-offs chấp nhận:** Cloud deployment (PDPA violation for pilot), không fine-tune
- **Trade-offs KHÔNG chấp nhận:** Latency >30s, hallucination rate >5%

**Why:** Cần rõ ràng về những gì team sẵn sàng compromise và những gì là red line.

#### Data Requirements — Specific & Owned

**Bản A:** "Course materials từ Intro to Programming" — chung chung.

**Bản B:**
- **Specific:** Slide PDF (Introduction, Control Flow, Functions, Arrays, Pointers), Lab 1-10, HW 1-5, past exams
- **Owner:** CS101 lecturer và TAs (ai upload, ai maintain)
- **Update frequency:** Mỗi học kỳ + pilot: upload 1 lần, no updates
- **Data format:** PDF → text extraction, code snippets giữ formatting, chunk theo logical sections
- **Rủi ro & mitigation:** Outdated materials → weekly sync với TA; OCR errors → manual review; incomplete coverage → track uncovered queries

**Why:** Data quality là backbone của RAG. Cần biết ai chịu trách nhiệm, update khi nào, và rủi ro cụ thể.

#### Fallback UX — Multi-layered, Realistic

**Bản A (DISASTER):**
- Trigger: `Confidence score < 80%` — **LLMs không có self-awareness**
- Hành động: "A TA will respond within 2 hours during business hours" — **false promise** vì students stuck tại 10 PM Saturday

**Bản B (REALISTIC):**

**Chiến lược:** Multi-layered fallback dựa trên **retrieval quality**, không phải model confidence.

**Level 1 — No retrieval (hit rate <30%):**
- "I couldn't find relevant material"
- Provide: Course FAQ link, Stack Overflow tag link, "Ask a TA" email form (NO SLA)
- **KHÔNG** nói "TA will respond within X hours"

**Level 2 — Low-quality retrieval (similarity <0.7):**
- Show answer với warning banner: "I'm not fully confident. Please double-check."
- Thêm nút: "Show me the source slide"
- Thêm nút: "This helped / This didn't help" (implicit feedback)

**Level 3 — Explicit negative feedback (thumbs down):**
- "Sorry about that. Would you like to: Try rephrasing / See similar questions / Contact a human TA"

**User options luôn có:**
- Ask again, Browse course materials (external link), Copy code

**Graceful degradation:**
- API fails → "Service temporarily unavailable"
- Retrieval slow → loading animation với message "This may take up to 30 seconds"

**Why this works:**
- ✅ Không dựa vào confidence score (impossible)
- ✅ Không hứa hẹn TA response (operationally impossible)
- ✅ Có multiple layers dựa trên metrics khả thi (retrieval hit rate, explicit feedback)
- ✅ Cung cấp self-service options — student vẫn có thể tự giải quyết

---

### 3. HYPOTHESIS TABLE

#### Hypothesis 1 — RAG Accuracy (Revised)

**Bản A:**
> "RAG hit rate >70% và user satisfaction score >4.0/5.0"

**Bản B:**
> "RAG accuracy ≥70% và **bug resolution rate ≥60%**"

**Why:** Satisfaction score là vanity metric (low participation). Bug resolution rate là **outcome** — student thực sự fix được bug.

**Test method:**
- Bản A: Manual QA với 50 câu hỏi
- Bản B: Real task với 10 volunteers, TA blind-review, measure bug resolution

---

#### Hypothesis 2 — Adoption (Painted Door Test)

**Bản A (MOM TEST FAILURE):**
> "Find 50 students, demo chatbot, hỏi 'Bạn có dùng nếu nó có trong Canvas không?'"

**Problem:** Hỏi hypothetical question → people say Yes để lịch sự. Không đo được actual behavior.

**Bản B (REAL BEHAVIOR):**
> "Painted Door Test: Static button 'Stuck? Ask the 24/7 AI TA' → click → modal 'We are at capacity. Enter email to join waitlist.'"

**Metrics:**
- Click-through rate (CTR) ≥20%
- Waitlist conversion ≥50%

**Why:** Đo **intent** qua behavior (click) thay vì stated intent. Industry benchmark: 2-5% CTR là typical, 20% là strong signal.

---

#### Hypothesis 3 — Fallback UX (Retrieval-based)

**Bản A:** Confidence score trigger (không khả thi)

**Bản B:** Retrieval hit rate trigger:
```
IF (retrieval_hit_rate < 0.3) OR (top_chunk_similarity < 0.7)
THEN activate fallback
```

**Success criteria:**
- Complaint rate ≤10%
- Retention after fallback ≥30%
- Session continuation ≥40%

---

#### Hypothesis 4 — PMF Signal (Task Resumption)

**Bản A:** Aha Moment = click citation + thumbs up (vanity)

**Bản B:** Aha Moment = **task resumption** — student hỏi bug, nhận answer, KHÔNG hỏi follow-up về cùng bug trong 15 phút.

**Why:** Behavior signal — student tiếp tục làm bài → thực sự unblocked.

**PMF Method:** Task Resumption Rate (TRR) ≥50% + Sean Ellis ≥40% + D7 retention ≥30%

---

### 4. PMF SCORECARD

#### Aha Moment — From Vanity to Behavior

| | Bản A | Bản B |
|---|-------|-------|
| **Definition** | Click citation + thumbs up + quay lại dùng lần 2 | Không hỏi follow-up về cùng bug trong 15 phút |
| **Metric** | (Sessions meeting 3 criteria) / Total | Task Resumption Rate (TRR) |
| **Reason** | Click ≠ đọc, thumbs up low participation, stressed users không rate | Behavior signal — tiếp tục làm bài = unblocked |

#### Vanity Metrics Avoided

**Bản A còn dùng:**
- Total queries
- Total sign-ups
- Pageviews
- Average response time

**Bản B explicitly forbid:**
- Tất cả trên + thêm:
  - Click-through rate on citations
  - Thumbs up/down ratio
  - Session duration
  - Number of materials uploaded

---

## KEY INSIGHTS FROM AI FEEDBACK

### 1. "Academic Exercise vs. Trenches"

**Problem:** Bản A viết như student assignment — có vẻ professional nhưng thiếu operational realism.

**Example:** "TA will respond within 2 hours during business hours" — nghe có vẻ responsible, nhưng **bất khả thi** vì:
- Ai trả lời TA queue lúc 10 PM Saturday?
- Tại sao students cần chatbot nếu fallback vẫn cần chờ đến Monday?

**Fix:** Fallback chỉ self-service. Nếu cần TA, students gửi email — nhưng **không hứa hẹn timeframe**.

---

### 2. "Mom Test Failure"

**Problem:** Hỏi hypothetical questions:
- "Bạn có dùng nếu nó có trong Canvas không?"
- "Bạn sẽ thất vọng nếu không còn dùng không?"

**Why fail:** People nói Yes để lịch sự, hoặc nói họ sẽ thất vọng vì听起来 hợp lý — nhưng behavior thực tế khác.

**Fix:** Đo **observed behavior**:
- Painted Door: Click button (intent signal)
- Actual usage: Query logs (adoption)
- Task resumption: Session analysis (value)

---

### 3. "Confidence Score is a Myth"

**Problem:** Dựa vào `confidence score < 80%` để trigger fallback.

**Reality:** GPT-4o sẽ **confidently hallucinate** — không có self-awareness. Confidence score từ LLM không có ý nghĩa gì.

**Fix:** Trigger dựa trên:
- **Retrieval quality:** Hit rate <30% hoặc similarity <0.7
- **Explicit feedback:** User click "This answer is wrong"
- **Pattern matching:** Query chứa từ khóa "error", "bug", "not working"

---

### 4. "Vanity Metric Trap"

**Problem:** Đo metrics trông đẹp nhưng không phản ánh value:
- Click citation → student tò mò, không có nghĩa là hiểu/trust
- Thumbs up → participation rate thấp, selection bias
- Total queries → có thể do 1 user spam

**Fix:** Đo **actionable metrics**:
- Task resumption → student tiếp tục làm bài = unblocked
- Bug resolution rate → student thực sự fix được bug
- Sean Ellis retention → user thực sự dependent on product

---

### 5. "Scope Creep: The Silent Killer"

**Problem:** In-Scope có 3 tính năng, trong đó:
- LMS integration — massive blocker
- Response time SLA — không phải feature

**Fix:** MVP Boundary phải chỉ giữ **1-2 killer features**. Các thứ khác là "có thể làm sau" hoặc "implementation detail".

**Checklist:**
- ✅ Mỗi In-Scope feature test đúng giả định nào?
- ✅ Cắt thêm 1 feature nữa có làm mất core value không?
- ✅ Out-of-Scope dài hơn In-Scope (chứng tỏ đã nói KHÔNG đủ)

---

## THAY ĐỔI LỚN NHẤT

| Category | Bản A | Bản B |
|----------|-------|-------|
| **Mindset** | "Build a complete product" | "Test the riskiest assumption with minimal effort" |
| **MVP Scope** | 3 In-Scope features (some nice-to-have) | 1-2 core features only |
| **Testing** | Surveys và stated intent (invalid) | Real behavior tracking (valid) |
| **Fallback** | False promises (TA within 2h) | Self-service only, no SLA |
| **PMF Metrics** | Vanity (clicks, ratings) | Actionable (task completion, retention) |

---

## RỦI RO CÒN LẠI SAU KHI SỬA

1. **Task resumption tracking:** Cần logging infrastructure (session logs, bug clustering). Có thể phức tạp để implement trong pilot 4 tuần.
   - **Alternative:** Dùng proxy đơn giản hơn: `time-to-next-query` — nếu student hỏi query khác sau 5-10 phút → có thể đã unblocked.

2. **Retrieval accuracy ≥70%:** Nếu course materials chất lượng thấp (OCR errors, outdated), hit rate sẽ thấp → fallback rate cao → user frustration.
   - **Mitigation:** Manual QA trước khi pilot, TA review materials.

3. **Painted Door CTR ≥20%:** Nếu CTR thấp (<10%) → có thể value proposition không đủ strong.
   - **Mitigation:** Refine messaging, test multiple value props.

4. **Sean Ellis cho educational tool:** Có phù hợp không? Students có thể nói "Very disappointed" vì mất quyền truy cập chứ không phải vì product value.
   - **Mitigation:** Phrasing câu hỏi cẩn thận: "How disappointed would be if you couldn't use the AI TA for your programming labs?" (specify context).

---

**Tổng kết:** Bản B là **10x better** vì:
1. Operational realism — không có false promises
2. Behavior-based testing — không có Mom Test
3. Fallback UX khả thi — không dựa vào magic confidence score
4. PMF metrics actionable — đo value, không đo vanity
5. MVP truly minimal — chỉ test core assumption
