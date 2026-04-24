# Hypothesis Table — Revised (Bản B)

**PRINCIPLE:** Mỗi hypothesis phải test bằng **actual behavior**, không phải stated intent. Không hỏi "Bạn có dùng không?" — đo **click, query, retention**.

---

## Hypothesis 1 — Course-grounded RAG Accuracy

> **"Chúng tôi tin rằng course-grounded RAG sẽ giúp first-year CS students đạt được accurate, trustworthy answers.**  
> **Chúng tôi sẽ biết mình đúng khi thấy RAG accuracy ≥70% trên 50 câu hỏi thực tế từ students, và students successfully resolve their bugs trong 60% cases."**

**Riskiest assumption phía sau hypothesis này:**
> Students sẽ trust và dùng chatbot nếu câu trả lời đúng với course materials và có citations, với accuracy đủ cao để giải quyết bugs.

**Cách test assumption này với chi phí thấp nhất (REAL TEST, không phải manual QA):**

1. **Setup:** Build simple RAG system với 100% CS101 materials uploaded.
2. **Recruit:** 10 volunteers từ CS101 class (không phải entire class — chỉ volunteers)
3. **Task:** Give them 3 real lab bugs từ assignments yêu cầu fix trong 15 phút mỗi bug.
4. **Measure:**
   - **Accuracy:** % câu trả lời đúng (do TA blind-review)
   - **Bug resolution rate:** % bugs được fix sau khi dùng chatbot (self-reported + TA verification)
   - **Time saved:** So với baseline (debug không có chatbot)
5. **Success criteria:**
   - Accuracy ≥70%
   - Bug resolution rate ≥60%
   - Time saved ≥30%

**Why this is a real test:** Students đang thực sự stuck và cần help — không phải hypothetical survey.

---

## Hypothesis 2 — Access & Adoption (Painted Door Test)

> **"Chúng tôi tin rằng low-friction access (standalone web app với SSO) sẽ giúp first-year CS students adopt chatbot với adoption rate ≥60% measured by actual usage.**  
> **Chúng tôi sẽ biết mình đúng khi thấy ≥60% students trong pilot class ít nhất 1 query trong 2 tuần đầu."**

**Riskiest assumption phía sau hypothesis này:**
> Students sẽ dùng chatbot nếu nó dễ access (single click, no signup), không cần LMS integration.

**Cách test assumption này với chi phí thấp nhất (Painted Door Test — KHÔNG PHẢI SURVEY):**

1. **Setup:**
   - Tạo static landing page với button lớn: "Stuck? Ask the 24/7 AI TA"
   - Button này **không mở chatbot** — mở modal: "We're preparing the system. Enter your email để được thông báo khi sẵn sàng."
   - Track: How many students click button, how many leave email.

2. **Deploy:**
   - Put button trên:
     - Course forum pinned post
     - TA office hours announcement
     - Weekly newsletter
   - Promote trong 1 tuần

3. **Measure:**
   - **Click-through rate (CTR):** % students trong class click button
   - **Waitlist conversion:** % clickers leave email
   - **Real intent signal:** Nếu CTR ≥20% → có real demand
     *(Industry benchmark: 2-5% là typical, 20% là strong signal)*

4. **Success criteria:**
   - CTR ≥20% trong 1 tuần
   - Waitlist conversion ≥50% của clickers

**Why this beats the old test:**
- Old test: "Bạn có dùng nếu nó có trong Canvas không?" → Mom Test failure (people say Yes to be polite)
- New test: **Observed behavior** (click) vs stated intent → valid signal

**Next step sau Painted Door:**
- Nếu CTR ≥20% → proceed to build chatbot
- Nếu <10% → rethink value proposition

---

## Hypothesis 3 — Fallback UX Acceptability

> **"Chúng tôi tin rằng fallback UX chỉ cung cấp self-service options (không hứa hẹn TA) sẽ được students chấp nhận khi chatbot không trả lời được.**  
> **Chúng tôi sẽ biết mình đúng khi thấy ≥70% fallback interactions không có complaint, và ≥30% users quay lại dùng chatbot sau fallback."**

**Riskiest assumption phía sau hypothesis này:**
> Students sẽ không complain nếu fallback chỉ cho "Browse materials" và "Ask again", không có live TA support.

**Cách test assumption này với chi phí thấp nhất:**

1. **Simulate fallback:** Trong pilot, intentionally make chatbot fail trên 10% queries (ví dụ: query về topic chưa có materials).
2. **Trigger fallback UX** như designed.
3. **Measure:**
   - **Complaint rate:** % fallback interactions có follow-up email/complaint
   - **Retention after fallback:** % users sau fallback, vẫn dùng chatbot lần tiếp theo trong 24h
   - **Session continuation:** % users sau fallback, vẫn tiếp tục chat (ask another question)
4. **Success criteria:**
   - Complaint rate ≤10%
   - Retention after fallback ≥30%
   - Session continuation ≥40%

**Why this matters:**
- Nếu students complain nhiều sau fallback → fallback UX fail → cần improve retrieval coverage
- Nếu students bỏ cuộc sau fallback → chưa giải quyết được Need #1

---

## Hypothesis 4 — Task Completion & PMF Signal

> **"Chúng tôi tin rằng students đạt Aha Moment khi chatbot successfully unblock họ trong debugging session.**  
> **Chúng tôi sẽ biết mình đúng khi thấy ≥50% queries dẫn đến task resumption (student tiếp tục code sau khi nhận answer) và ≥40% users trong pilot trả lời 'Very disappointed' nếu không còn dùng chatbot."**

**Riskiest assumption phía sau hypothesis này:**
> Real PMF signal cho educational tool là **task completion**, không phải satisfaction scores hay click metrics.

**Cách test assumption này:**

1. **Track task resumption:**
   - Khi student gửi query về bug, đánh dấu session đó.
   - Sau 5-15 phút, check xem student có gửi query mới về cùng bug không (follow-up) → Nếu không, có thể đã fix.
   - **Proxy:** Time between query và next query about different topic (session continuation).
2. **Sean Ellis Test:** Sau 4 tuần, survey:
   - "How would you feel if you could no longer use the TA chatbot?"
   - Options: Very disappointed, Somewhat disappointed, Not disappointed, N/A
   - Ngưỡng: ≥40% "Very disappointed"
3. **Retention curve:** D7 retention — % students sau 7 ngày vẫn dùng (ít nhất 1 query/tuần).
4. **Success criteria:**
   - Task resumption rate ≥50% (proxy: no follow-up query on same bug within 15 min)
   - Sean Ellis: ≥40% "Very disappointed"
   - D7 retention ≥30%

**Why this beats old Aha Moment:**
- Old: "Click citation + thumbs up" — vanity metric (click ≠ value, thumbs up low participation)
- New: **Behavioral metric** — student tiếp tục làm bài → thực sự unblocked
