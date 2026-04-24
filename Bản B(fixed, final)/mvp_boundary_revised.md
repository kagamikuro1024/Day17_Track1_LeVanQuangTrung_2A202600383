# MVP Boundary Sheet — Revised (Bản B)

**Riskiest Assumption (Sau AI Stress-Test):**
> Sinh viên sẽ thực sự dùng chatbot khi họ bị stuck với bug, với adoption rate ≥60% measured by actual usage (not survey), và RAG accuracy ≥70% trên câu hỏi thực tế.

**Lý do đổi từ "≥80% adoption" sang "≥60% actual usage":**
- AI feedback chỉ ra: hỏi "Bạn có dùng không?" là Mom Test failure — students sẽ nói Yes để lịch sự
- Cần đo **actual behavior** (click button, send query) thay vì stated intent

---

**IN-SCOPE** (tính năng cốt lõi bắt buộc để test giả thuyết) — **ĐÃ CẮT CHỈ CÒN 2 TÍNH NĂNG**:

- [ ] **Course-grounded RAG Q&A** — Hệ thống retrieval từ course materials (slides, labs, assignments) để trả lời câu hỏi programming với citations
  - **Test giả định:** Students sẽ trust và dùng chatbot nếu câu trả lời đúng với khóa học và có nguồn rõ ràng
  - **Why essential:** Đây là killer feature duy nhất — nếu không có course-specific answers, thì khác gì ChatGPT?

- [ ] **Standalone web app (hoặc Discord bot)** — Giao diện chat độc lập, không cần LMS integration
  - **Test giả định:** Access low-friction sẽ tăng adoption; students có thể access từ phone/laptop mà không cần login Canvas
  - **Why essential:** LMS integration là massive blocker (IT approvals, security reviews) — không cần thiết để test core value

---

**OUT-OF-SCOPE** (tính năng tốt nhưng không cần cho MVP) — **ĐÃ THÊM**:

- [ ] **Chat interface tích hợp vào LMS** — ĐÃ CHUYỂN SANG OUT-OF-SCOPE
  - Lý do: Massive blocker — IT approvals, security reviews, weeks của bureaucratic red tape
  - Alternative: Standalone web app với SSO (single sign-on) bằng university account là đủ
- [ ] **Response time ≤5 phút** — ĐÃ CHUYỂN SANG OUT-OF-SCOPE
  - Lý do: Đây là SLA/non-functional requirement, không phải feature bạn "build"
  - Với GPT-4o + basic vector DB, latency sẽ tự nhiên <10s — không cần list như In-Scope
- [ ] **ReAct agent với tool calling** — Chỉ dùng simple RAG retrieval
- [ ] **Hybrid deployment** — Bắt đầu với cloud deployment để test adoption nhanh
- [ ] **TA dashboard & analytics** — Chỉ focus vào student experience
- [ ] **Mobile app** — Responsive web app là đủ
- [ ] **Voice input/output** — Text only

---

**NON-GOALS** (ranh giới đỏ — sản phẩm sẽ KHÔNG làm) — **ĐÃ THÊM**:

- [ ] **Hỗ trợ môn học năm 2, 3, 4** — Chỉ first-year courses
- [ ] **Hỗ trợ môn không phải CS** — Chỉ programming labs
- [ ] **Fine-tune model riêng** — Dùng off-the-shelf LLM với RAG
- [ ] **Tự động update course materials** — TA/lecturer upload manually
- [ ] **Xử lý PII data trong MVP** — Anonymous queries only
- [ ] **Revenue generation** — Internal tool, không có pricing
- [ ] **24/7 TA support via fallback** — **QUAN TRỌNG:** Fallback không hứa hẹn TA response tối/cuối tuần
  - Lý do: Sẽ tạo expectation rồi fail — chính là problem ban đầu
  - Alternative: Fallback chỉ cung cấp "Browse course materials" và "Ask again" — không hứa hẹn TA

---

**Checkpoint 1 — Kill questions (REVISED):**

1. **Nếu cắt thêm 1 tính năng trong In-Scope, khách hàng còn nhận được giá trị cốt lõi không?**
   - Cắt RAG Q&A → không còn value (chatbot becomes generic)
   - Cắt standalone web app → có thể dùng CLI/terminal? Không, cần accessible UI
   → **Kết luận:** Chỉ nên giữ 1 tính năng duy nhất là **RAG Q&A**, standalone web app là delivery mechanism có thể thay thế.

2. **In-Scope có nhiều hơn 3 tính năng không?**
   - **Bản cũ:** 3 (quá nhiều)
   - **Bản mới:** 2 (có thể cắt xuống 1)
   - **Recommendation:** Giữ **chỉ RAG Q&A** là In-Scope, standalone web app là implementation detail có thể Out-of-Scope nếu cần.

3. **Non-Goals có ít nhất 1 thứ team đang rất muốn làm không?**
   - **Yes:** LMS integration (vì nghe có vẻ chuyên nghiệp)
   - **Yes:** 24/7 TA fallback (vì nghe có vẻ an toàn)
   - **Yes:** Hybrid deployment (vì compliance)
   → **OK.**
