# PMF Scorecard — Revised (Bản B)

Dựa trên AI feedback và best practices từ các case studies (Facebook, Twitter, Spotify), tôi đã thay đổi hoàn toàn cách tiếp cận PMF metrics.

---

## Aha Moment — Tái định nghĩa

**Aha Moment CHO STUDENT LÀ GÌ?**

Trong educational context, Aha Moment không phải là "click citation" hay "nhận rating". Đó là:
> **Khoảnh khắc student thực sự giải quyết được problem và có thể tiếp tục làm bài.**

Hành vi cụ thể:
1. Student hỏi về bug/error
2. Nhận câu trả lời
3. **Không hỏi follow-up về cùng bug** (implicit success signal)
4. **Quay lại làm bài** (tiếp tục coding, compile, run)
5. **Có thể hỏi query khác sau đó** (continuation signal)

**OLD (vanity):** "Click citation link + thumbs up"
- Problem: Click ≠ đọc, thumbs up participation rate thấp, stressed students không rate
**NEW (actionable):** "Task resumption + no follow-up on same bug"

---

## Actionable Metrics

### Primary Metric: **Task Resumption Rate (TRR)**

> **Definition:** % của queries mà student KHÔNG hỏi follow-up question về cùng bug/issue trong 15 phút sau.

**Logic:**
- Nếu chatbot trả lời đúng và hữu ích → student sẽ tiếp tục làm bài (không cần hỏi lại)
- Nếu chatbot trả lời sai/không hữu ích → student sẽ hỏi lại (same bug, rephrased) trong 15 phút
- 15 phút là window đủ để student test answer và quyết định có cần hỏi lại không

**Cách đo:**
1. Đánh dấu mỗi query với `session_id` và `bug_id` (detect same bug qua semantic similarity)
2. Track xem có query nào trong 15 phút sau về cùng bug không
3. Nếu không → count là "resumed"
4. TRR = (Queries with no follow-up) / (Total queries)

**Ngưỡng thành công:**
- **Minimum:** TRR ≥ 50%
- **Target:** TRR ≥ 60%
- **Excellent:** TRR ≥ 70%

---

### Secondary Metrics (theo Sean Ellis Framework):

**1. Sean Ellis Test (40% Rule)**
- Câu hỏi: "How would you feel if you could no longer use the TA chatbot?"
- Options:
  - Very disappointed
  - Somewhat disappointed
  - Not disappointed
  - N/A (I no longer use it)
- **Ngưỡng PMF:** ≥40% "Very disappointed"

**2. D7 Retention (Educational app benchmark)**
- Definition: % students vẫn active (ít nhất 1 query/tuần) sau 7 ngày từ first use
- **Ngưỡng PMF:** D7 retention ≥ 30%
- Lý do: Educational tools có retention thấp hơn consumer apps vì usage theo học kỳ

**3. Adoption Rate (Actual behavior)**
- Definition: % students trong pilot class dùng chatbot ít nhất 3 lần trong 2 tuần đầu
- **Ngưỡng PMF:** Adoption ≥ 60%
- **Measurement:** Painted Door test trước, sau đó actual usage logs

---

## PMF Scorecard Table

| Metric | Definition | Measurement Method | Ngưỡng PMF | Frequency |
|--------|------------|-------------------|------------|-----------|
| **Task Resumption Rate (PRIMARY)** | % queries không có follow-up về cùng bug trong 15 phút | Session logs + bug clustering (semantic similarity) | ≥ 50% | Weekly |
| **Sean Ellis Score** | % users "Very disappointed" nếu mất sản phẩm | Post-pilot survey (4-week) | ≥ 40% | Once (end of pilot) |
| **D7 Retention** | % students active sau 7 ngày | Cohort analysis từ usage logs | ≥ 30% | Weekly |
| **Adoption Rate** | % students dùng ≥3 lần trong 2 tuần | Usage logs per class roster | ≥ 60% | Weekly |
| **RAG Accuracy** | % câu trả lời đúng (TA blind review) | Manual QA: 50 random queries | ≥ 70% | Weekly during pilot |
| **Retrieval Hit Rate** | % queries có retrieval hit (similarity >0.7) | Vector DB metrics | ≥ 70% | Daily |

---

## Vanity Metrics — KHÔNG DÙNG

 **Total queries** — Có thể do 1 user spam
 **Total users** — Không biết họ có get value không
 **Average response time** — Chỉ đo hệ thống, không đo user satisfaction
 **Click-through rate on citations** — Click ≠ value
 **Thumbs up/down ratio** — Low participation, selection bias
 **Session duration** — Dài không có nghĩa là helpful (có thể user đang frustrated)
 **Number of course materials uploaded** — Input metric, không đo output

---

## PMF Method & Timeline

**Primary PMF Method:** **Task Resumption Rate** + **Sean Ellis Test**

**Why this combination:**
- **Task Resumption:** Leading indicator — đo hành vi thực tế, có thể track ngay từ ngày 1
- **Sean Ellis:** Lagging indicator — cần đủ users sau 4 tuần để survey có meaning
- **Retention:** Supporting metric — D7 để see if students quay lại

**Measurement timeline:**

| Phase | Duration | Metrics tracked | Decision point |
|-------|----------|-----------------|----------------|
| **Beta (Week 1-2)** | 2 tuần | TRR, Retrieval hit rate, RAG accuracy (manual QA), Adoption rate | Nếu TRR <40% hoặc Accuracy <60% → cần improve retrieval trước khi scale |
| **Pilot (Week 3-4)** | 2 tuần | TRR, D7 retention, Sean Ellis (Week 4), Adoption | Sau Week 4: Nếu TRR ≥50% và Sean Ellis ≥30% → proceed to expand. Nếu <30% → pivot or kill |
| **Post-pilot analysis** | 1 tuần | Full metrics, survey, user interviews | Final PMF determination |

**Go/No-Go criteria:**
- ✅ **GREEN:** TRR ≥50% AND Sean Ellis ≥35% AND D7 retention ≥25%
- 🟡 **YELLOW:** TRR 40-50% OR Sean Ellis 25-35% → cần iterate và test thêm 4 tuần
- 🔴 **RED:** TRR <40% OR Sean Ellis <25% → **Hypothesis invalidated** — rethink core value prop

---

## Aha Moment — Detection & Engineering

**How to detect Aha Moment in logs:**

1. **Pattern matching:**
   - User asks bug question → gets answer → NO follow-up on same bug within 15 min → session continues with different topic → **count as Aha**
   - User asks bug question → gets answer → follow-up same bug within 15 min → **NOT Aha** (answer didn't resolve)

2. **Aha Moment Rate (AMR):**
   ```
   AMR = (Users with ≥1 Aha session in first 3 sessions) / (Total new users)
   ```
   - First 3 sessions: critical window để user đánh giá value
   - Ngưỡng: AMR ≥ 60% → strong PMF signal

3. **Engineering the Aha Moment:**
   - Nếu AMR thấp (<40%), xem:
     - Retrieval accuracy có đủ không?
     - Câu trả lời có actionable không (step-by-step debugging)?
     - Citations có đủ rõ ràng để student verify?
   - Build features để **increase Aha probability**:
     - Pre-built debugging templates (common errors)
     - Code snippet with explanation
     - Visual diagrams (nếu cần)

---

## PMF Scorecard Summary

**Aha Moment (behavioral definition):**
> Student asks a bug-related question, receives answer, and **does NOT ask a follow-up about the same bug within 15 minutes**, indicating the answer successfully unblocked them.

**Actionable Metric (Primary):**
> **Task Resumption Rate (TRR)** = % queries with no follow-up on same bug within 15 minutes. Target: ≥50%.

**PMF Method:**
> - **Sean Ellis Test** (≥40% "Very disappointed")
> - **D7 Retention** (≥30%)
> - **Adoption Rate** (≥60% use ≥3 times in 2 weeks)

**Vanity Metrics avoided:**
> Total queries, clicks, pageviews, satisfaction scores (without behavior), average response time.

**Measurement readiness:**
- Need logging: `user_id`, `query`, `timestamp`, `retrieval_chunks`, `response`
- Need clustering algorithm: Detect same bug via semantic similarity (cosine similarity >0.8 trên query embeddings)
- Need manual QA pipeline: 50 random queries/week do TA blind review