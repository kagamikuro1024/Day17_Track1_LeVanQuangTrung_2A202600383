# README — Day 17: PRD & Product-Market Fit

## 🎯 Mục tiêu Day 17

Chuyển từ **strategy** (Day 16) sang **executable product plan** — PRD có thể đưa cho engineer để bắt đầu build.

**Central question:** Làm sao chuyển hóa chiến lược đúng thành sản phẩm có thể đo lường được?

---

## 📦 Deliverables (5 files)

### 1. MVP Boundary Sheet
**Câu hỏi:** Build gì trước, bỏ gì lại?

- **In-Scope:** Tính năng cốt lõi bắt buộc để test giả định
- **Out-of-Scope:** Tính năng tốt nhưng không cần cho MVP
- **Non-Goals:** Ranh giới đỏ — sản phẩm sẽ KHÔNG làm

**Killer principle:** MVP là bài test rẻ nhất để mua sự thật từ thị trường — không phải sản phẩm thiếu tính năng.

**Key insight:** In-Scope nên chỉ có **1-2 tính năng**. Nếu có 5+ → bạn chưa sắc.

---

### 2. PRD Skeleton (Product Requirements Document)

**PRD là gì?** Văn bản đồng thuận về "Cái gì" và "Tại sao" — không phải "Làm thế nào".

**6 thành phần standard:**
1. Problem Statement — Ai đang gặp vấn đề gì? Tác động kinh tế?
2. Target User — Ai là người dùng ưu tiên?
3. User Stories — As a [role], I want [action] so that [outcome]
4. MVP Scope — In/Out/Non-goals (đã làm ở WS1)
5. Success Metrics — Đo lường thành công thế nào?
6. Dependencies & Constraints — Ràng buộc hệ thống và nguồn lực

**3 thành phần AI-specific (BẮT BUỘC với AI product):**
7. Model Selection Rationale — Tại sao chọn model này? Trade-offs?
8. Data Requirements — Nguồn dữ liệu cho RAG/fine-tuning? Ai sở hữu? Update frequency?
9. Fallback UX — Khi AI sai/không tự tin → hệ thống làm gì?

**⚠️ Critical:** Fallback UX là điểm khác biệt lớn nhất giữa AI PRD và PRD truyền thống.

---

### 3. Hypothesis Table

**Mọi tính năng trong PRD đều là một vụ cá cược.**

**Công thức:**
> Chúng tôi tin rằng [Tính năng X] sẽ giúp [Nhóm khách hàng Y] đạt được [Kết quả Z].  
> Chúng tôi sẽ biết mình đúng khi thấy [Metric M] đạt [Ngưỡng T].

**RAT (Riskiest Assumption Test):**
Tìm giả định nguy hiểm nhất — nếu sai thì toàn bộ dự án sụp đổ — và test nó trước tiên.

**Key principle:** Hypothesis phải **sai được**. Nếu bạn không thể nghĩ ra cách hypothesis này bị falsified — nó quá mơ hồ.

---

### 4. PMF Scorecard

**PMF không phải là cảm giác. PMF là con số.**

**3 cách đo PMF:**

1. **Sean Ellis Test (40% Rule):**
   - "Bạn sẽ cảm thấy thế nào nếu không còn dùng được sản phẩm này?"
   - Ngưỡng: >40% "Very disappointed"

2. **Retention Curve:**
   - Đường cong giữ chân (cohort analysis)
   - Ngưỡng: D30 >10% (B2C), D30 >30% (B2B SaaS)

3. **Aha Moment / Magic Number:**
   - Tìm hành vi cụ thể mà retained users đều làm trong ngày đầu
   - Ví dụ: Facebook — "7 friends in 10 days"

**⚠️ Vanity vs Actionable Metrics:**

| Vanity Metrics (ĐỪNG DÙNG) | Actionable Metrics (DÙNG) |
|---------------------------|---------------------------|
| Total downloads | D30 Retention Rate |
| Total sign-ups | % users reaching Aha Moment |
| Pageviews | Task completion rate |
| Revenue tổng | Revenue per active user |
| DAU/MAU tổng | DAU/MAU ratio (stickiness) |

**Rule:** Nếu metric tăng mà hành vi cốt lõi không đổi → đó là vanity metric.

---

## 🔍 Key Insights from AI Stress-Test

### 1. "Academic Exercise vs. Trenches"

**Anti-pattern:** Viết PRD như bài tập học thuật — professional nhưng thiếu operational realism.

**Example:** "TA will respond within 2 hours" — nghe có vẻ responsible, nhưng **bất khả thi** vì ai trả lời TA queue lúc 10 PM Saturday?

**Fix:** Không hứa hẹn SLA cho fallback nếu không chắc chắn có thể deliver.

---

### 2. "Mom Test Failure"

**Anti-pattern:** Hỏi hypothetical questions:
- "Bạn có dùng nếu nó có trong Canvas không?"
- "Bạn sẽ thất vọng nếu không còn dùng không?"

**Why fail:** People nói Yes để lịch sự. Stated intent ≠ actual behavior.

**Fix:** Đo **observed behavior**:
- Painted Door Test (click tracking)
- Actual usage logs
- Session analysis

---

### 3. "Confidence Score is a Myth"

**Anti-pattern:** Dựa vào `confidence score < 80%` để trigger fallback.

**Reality:** LLMs **confidently hallucinate** — không có self-awareness. Confidence score từ model không có ý nghĩa.

**Fix:** Trigger dựa trên:
- Retrieval quality metrics (hit rate, similarity)
- Explicit user feedback ("This answer is wrong")
- Pattern matching (query contains "error", "bug")

---

### 4. "Vanity Metric Trap"

**Anti-pattern:** Đo metrics trông đẹp nhưng không phản ánh value:
- Click citation → tò mò, không có nghĩa là trust
- Thumbs up → participation rate thấp
- Total queries → có thể do spam

**Fix:** Đo **actionable metrics**:
- Task resumption (student tiếp tục làm bài)
- Bug resolution rate (student fix được bug)
- Retention (user quay lại)

---

### 5. "Scope Creep: The Silent Killer"

**Anti-pattern:** In-Scope có 5+ tính năng, nhiều nice-to-have.

**Fix:** MVP Boundary phải chỉ giữ **1-2 killer features**. Dùng kill question:
> Nếu cắt thêm 1 tính năng trong In-Scope, khách hàng còn nhận được giá trị cốt lõi không?

Nếu **CÓ** → tính năng đó không phải core, chuyển sang Out-of-Scope.

---

## 📋 Checklist Before Submit

### MVP Boundary
- [ ] In-Scope ≤ 3 tính năng (ideal: 1-2)
- [ ] Mỗi In-Scope feature test đúng giả định nào?
- [ ] Out-of-Scope dài hơn In-Scope (chứng tỏ đã nói KHÔNG đủ)
- [ ] Non-Goals có ranh giới đỏ rõ ràng (team đang muốn làm)

### PRD Skeleton
- [ ] User Stories mô tả hành vi, không mô tả UI
- [ ] Fallback UX có trigger cụ thể và hành động cụ thể
- [ ] Model Selection có lý do và trade-offs (không chỉ tên model)
- [ ] Data Source có tên cụ thể (không "từ internet")
- [ ] Dependencies & Constraints đầy đủ (API, timeline, budget, legal)

### Hypothesis Table
- [ ] Mỗi In-Scope feature có 1 hypothesis
- [ ] Hypothesis có metric và ngưỡng rõ ràng
- [ ] Test method là **actual behavior**, không phải survey
- [ ] Riskiest assumption được identify và test với chi phí thấp

### PMF Scorecard
- [ ] Aha Moment là hành vi cụ thể (không phải cảm giác)
- [ ] Primary metric là actionable (không vanity)
- [ ] PMF method phù hợp với loại sản phẩm
- [ ] Vanity metrics được liệt kê (để tránh)

---

## 🎓 Bài học lớn nhất

**"Build products, not assignments."**

Một PRD tốt:
- ✅ Có thể đưa cho engineer và họ hiểu ngay build cái gì
- ✅ Có thể đưa cho designer và họ vẽ được user flow
- ✅ Có thể đưa cho stakeholder và họ hiểu tại sao làm
- ✅ Có thể đưa cho user và họ dùng được (vì đã test với real behavior)

Một PRD tồi:
- ❌ Viết đẹp nhưng không actionable
- ❌ Có metrics nhưng không đo được value thực
- ❌ Có fallback nhưng bất khả thi operationally
- ❌ Có scope nhưng không test đúng risk

---

## 📚 Quick Reference

| Khi nào nhớ câu nào |
|---------------------|
| "MVP is the cheapest way to buy the truth from the market." |
| "The best MVP is the one that knows how to say NO." |
| "In traditional software, a button does what it's told. In AI, you must design for when it disobeys." |
| "The best Fallback UX is managing user expectations from second zero." |
| "PMF is not a feeling — it is an engineering process." |
| "Don't measure the busyness of the system, measure the behavior that creates core value." |
| "If a metric goes up but core behavior doesn't change — it's a vanity metric." |
| "A hypothesis must be falsifiable. If you can't imagine how it could be wrong — it's too vague." |

---

## 🚀 Next Steps

Sau Day 17, bạn sẽ có **4 tài liệu**:
1. MVP Boundary Sheet
2. PRD Skeleton
3. Hypothesis Table
4. PMF Scorecard

**Đây là hành trang bước vào giai đoạn build thực tế.**

Hãy lưu kỹ — vì đây là **sổ tay tham chiếu** cho toàn bộ quá trình build sản phẩm.
