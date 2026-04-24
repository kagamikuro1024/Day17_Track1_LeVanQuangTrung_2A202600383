# PRD Skeleton — Revised (Bản B)

Dựa trên AI Stress-Test feedback, đã sửa các phần AI-specific và User Stories.

---

### Problem Statement

First-year CS students tại VinUniversity thường bị stuck với bug programming vào **tối 19h-23h và cuối tuần**, khi TA không có sẵn. Họ mất **2-4 giờ** mỗi lần để debug qua Google/StackOverflow, dẫn đến deadline anxiety, chất lượng bài nộp thấp, và muộn graduation.

**Tác động kinh tế:** Giảm 20 giờ/tuần TA workload → tiết kiệm ~$10,000/year cho CS department.

---

### Target User

**First-year CS students** (150-200 sv/khóa) tại VinUniversity, tuổi 18-20, đang học môn Intro to Programming (CS101). Họ:
- Làm bài tập vào tối và cuối tuần
- Có ít kinh nghiệm debug
- Cảm thấy panic khi gặp bug không giải thích được
- Thường tìm kiếm giải pháp nhanh trên Google/StackOverflow
- Có thể access internet từ laptop/phone

**Access method:** Standalone web app (https://ta-chatbot.vinuni.edu.vn) với single sign-on qua university account — **KHÔNG cần LMS integration**.

---

### User Stories (REVISED — không mô tả UI)

**Story 1:**
> As a first-year CS student, I want to describe my programming error in natural language and receive a step-by-step debugging guide within 60 seconds, so that I can fix the bug and continue my lab without waiting for TA office hours.

**Story 2:**
> As a first-year CS student, I want to see exactly which slide/page in my course materials supports the answer, so that I can cross-check with the professor's explanation and avoid learning incorrect concepts.

**Story 3:**
> As a first-year CS student, I want to rephrase my question if the first answer doesn't help, so that I can get a different explanation approach without starting from scratch.

---

### AI-Specific (REVISED)

**Model Selection:**

- **Model:** GPT-4o (hoặc Claude 3.5 Sonnet)
- **Lý do chọn:**
  - **Accuracy:** State-of-the-art cho programming debugging — code generation, error explanation, step-by-step reasoning
  - **Latency:** ~1-2s API call + retrieval ~2-3s → total <10s (đủ cho real-time help)
  - **Cost:** ~$0.01-0.05/query → với 200 students × 3 queries/ngày = ~$360-1,800/tháng → chấp nhận được cho pilot
  - **Instruction following:** Tốt để tuân theo prompt template và citation requirements
- **Trade-offs chấp nhận:**
  - Cloud deployment (vi phạm PDPA) — chấp nhận trong MVP để test adoption nhanh, sẽ migrate on-premise sau
  - Không fine-tuned — RAG là đủ cho domain specificity, fine-tune không cần thiết cho use-case này
  - Context window 128K — đủ cho 10-20 slides + query + response
- **Trade-offs KHÔNG chấp nhận:**
  - Latency >30s — user sẽ abandon
  - Hallucination rate >5% trong code examples — có thể cause syntax errors cho students

**Data Requirements (SPECIFIC):**

- **Nguồn dữ liệu:**
  - **Primary:** CS101 course materials từ VinUniversity:
    - Slide PDF (Introduction, Control Flow, Functions, Arrays, Pointers)
    - Lab specifications (Lab 1-10)
    - Assignment descriptions (HW 1-5)
    - Past exam questions (midterm, final)
  - **Secondary:** Common error patterns từ Stack Overflow (filtered by course topics)
- **Owner:** CS101 lecturer và TAs — họ upload và maintain knowledge base
- **Update frequency:**
  - Mỗi học kỳ: Upload materials cho khóa học mới
  - Giai đoạn pilot: Upload 1 lần, không update trong 4 tuần
- **Data format:**
  - PDF → text extraction (OCR không cần nếu đã text)
  - Code snippets trong slides giữ nguyên formatting
  - Mỗi document được chunk theo logical sections (lab, topic, concept)
- **Rủi ro về data quality:**
  1. **Materials outdated:** Lecturer thay đổi syllabus giữa học kỳ → retrieval trả lời sai
     - Mitigation: Weekly sync với TA để verify hit rate và accuracy
  2. **OCR errors:** Slide scan → text sai → retrieval sai
     - Mitigation: Manual review trước khi upload (TA 1-2h/work)
  3. **Incomplete coverage:** Student hỏi về topic không có trong materials → system không trả lời được
     - Mitigation: Track uncovered queries, weekly report cho lecturer để bổ sung materials

**Fallback UX (REVISED — KHÔNG dựa vào confidence score):**

**Vấn đề với bản cũ:** LLMs không có self-awareness — GPT-4o sẽ confidently hallucinate. Confidence score <80% không hoạt động.

**Chiến lược mới: Multi-layered fallback với trigger dựa trên retrieval quality, không phải model confidence.**

---

**Fallback UX Design:**

- **Chiến lược:** **Human-in-the-loop + Expectation Management kết hợp**
  - Level 1: Khi retrieval không tìm thấy relevant chunks (hit rate <30% hoặc top-5 similarity <0.7)
  - Level 2: Khi query chứa từ khóa "error", "bug", "not working" và retrieval still low quality
  - Level 3: Khi user click "This answer is wrong" (explicit feedback)

- **Trigger cụ thể:**
  ```
  IF (retrieval_hit_rate < 0.3) OR (top_chunk_similarity < 0.7)
  THEN activate fallback
  ```

- **Hành động (Level 1 — No retrieval):**
  - Hiển thị: "I couldn't find relevant material in your course slides. Here are some general suggestions:"
  - Provide:
    1. Link to official course FAQ page (nếu có)
    2. Link to Stack Overflow với tag phù hợp (ví dụ: `python`, `c-pointers`)
    3. Button: "Ask a TA" → mở form gửi email đến TA queue (không hứa hẹn response time)
  - **KHÔNG** nói "A TA will respond within 2 hours" vì có thể false promise

- **Hành động (Level 2 — Low-quality retrieval):**
  - Hiển thị câu trả lời nhưng với warning banner:
    ```
    ⚠️ I found some information, but I'm not fully confident.
    Please double-check with your course materials before using this answer.
    ```
  - Thêm nút: "Show me the source slide" — link trực tiếp đến slide được retrieval
  - Thêm nút: "This helped / This didn't help" — implicit feedback

- **Hành động (Level 3 — Explicit negative feedback):**
  - Khi user click "This answer is wrong" (thumbs down):
    - Hiển thị: "Sorry about that. Would you like to:"
    - Options:
      1. "Try rephrasing my question" — mở chat lại
      2. "See similar questions from other students" — hiển thị FAQ từ course
      3. "Contact a human TA" — mở form gửi email

- **User options luôn có:**
  - [Ask again] — retry với câu hỏi mới
  - [Browse course materials] — link trực tiếp đến LMS materials page (external link)
  - [Copy code] — nếu answer chứa code snippet

- **Graceful degradation:**
  - Nếu API fails hoặc rate limit: Hiển thị "Service temporarily unavailable. Please try again in 1 minute."
  - Nếu retrieval quá chậm (>10s): Show loading animation với message "Searching your course materials... This may take up to 30 seconds."

---

### Success Metrics

*(Điền chi tiết trong Hypothesis/PMF)*

- **Primary metric:** ___________________________________________
- **Secondary metrics:** ________________________________________
- **Ngưỡng thành công:** ________________________________________
- **Timeframe đo lường:** _______________________________________

---

### Dependencies & Constraints (REVISED)

- **API / Service cần tích hợp:**
  - LLM API: OpenAI GPT-4o hoặc Anthropic Claude 3.5 Sonnet
  - Vector DB: FAISS (self-hosted) hoặc Pinecone (cloud)
  - Auth: VinUniversity SSO (OAuth 2.0) — cần IT support để setup
  - Hosting: Vercel/Netlify cho web app (static deployment)

- **Timeline constraint:**
  - Pilot: 4 tuần trong học kỳ hiện tại
  - Setup infrastructure: 1 tuần
  - Upload materials: 3 ngày
  - Beta testing với 1 lớp (50 students): 1 tuần
  - Full pilot: 2 tuần
  - Analysis: 1 tuần

- **Budget constraint:**
  - API costs: $500-1,000/tháng (worst-case)
  - Vector DB: $0-100/tháng (FAISS free, Pinecone paid)
  - Hosting: $0-50/tháng (Vercel free tier đủ)
  - **Total:** < $1,500 cho 4-week pilot

- **Legal / Compliance constraint:**
  - Cloud deployment vi phạm PDPA → cần document này là **research/experiment only**, không production
  - Student data: Anonymous queries only — không lưu PII
  - Data retention: Delete logs sau 30 ngày
  - Requires approval từ VinUniversity CS department head và IT security (even for pilot)

- **Operational constraints:**
  - **NO 24/7 TA support commitment** — fallback chỉ cung cấp self-service options
  - TA workload: 1-2h/tuần để review materials và monitor accuracy (không cần triage fallback queue)
  - No SLA đối với chatbot — best effort only