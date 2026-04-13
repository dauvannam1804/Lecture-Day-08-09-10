# Group Report — Lab Day 08: RAG Pipeline

Day 08 — RAG Pipeline  
**Ngày nộp:** 2026-04-13  
Nguyễn Trí Cao, Đậu Năm Nam, Lê Minh Tuấn, Cao Diệu Ly



---

## 1) Quyết định kỹ thuật (cấp nhóm)

### 1.1 Indexing + Chunking
- **Chiến lược chunking:** Paragraph-based để tránh cắt giữa câu/điều khoản (giữ tính mạch lạc của SOP/policy).
- **Thông số:** `chunk_size = 400 tokens`, `overlap = 80 tokens` (cân bằng giữa “đủ ngữ cảnh” và “không quá dài” cho retrieval + generation).
- **Embedding model:** `text-embedding-3-small` (ổn định, chi phí hợp lý cho corpus nhỏ, phù hợp nhu cầu demo/lab).
- **Vector store:** ChromaDB (persistent) + cosine similarity.

### 1.2 Metadata schema (phục vụ retrieval + citation + filter)
Nhóm thống nhất chuẩn metadata tối thiểu để:
- Hỗ trợ **citation** rõ ràng (trả về `[1]`, `[2]` dựa trên `source/section`).
- Cho phép mở rộng **filter theo ngữ cảnh** (ví dụ department / effective_date).

**Các field chính đã dùng:** `source`, `section`, `effective_date`, `department`, `access`.

### 1.3 Retrieval: chọn Baseline làm cấu hình “best overall”
Nhóm triển khai 2 cấu hình và A/B theo scorecard:
- **Baseline (Sprint 2):** Dense retrieval (vector similarity).
- **Variant (Sprint 3):** Hybrid (Dense + BM25) và fusion bằng **RRF (Reciprocal Rank Fusion)**.

**Kết luận chọn cấu hình chạy “best” (ưu tiên khi grading):**
- Chọn **Baseline (dense)** làm cấu hình chính vì **điểm trung bình tốt hơn và ổn định hơn** trên bộ test nội bộ (đặc biệt `Relevance`/`Completeness`).
- Variant Hybrid giữ lại như **đường hướng tối ưu tiếp theo** cho các truy vấn có keyword/alias mạnh (ví dụ “Approval Matrix”, mã lỗi), nhưng cần thêm bước lọc nhiễu.

### 1.4 Generation: prompt grounded + nguyên tắc abstain
Nhóm thống nhất prompt theo nguyên tắc:
- **Chỉ trả lời dựa trên context**.
- **Thiếu context thì abstain** (không bịa), đồng thời vẫn trả lời ngắn gọn + có citation khi có thể.
- **Temperature = 0** để giảm dao động khi chấm điểm.

### 1.5 Evaluation: LLM-as-Judge để so sánh A/B
Nhóm dùng scorecard theo 4 metric: `Faithfulness`, `Relevance`, `Context Recall`, `Completeness`.  
Mục tiêu là ra quyết định dựa trên trade-off thực tế (không chỉ nhìn Recall).

---

## 2) Kết quả A/B và trade-off quan trọng

### 2.1 Scorecard tổng quan (từ `results/`)
- **Baseline (dense):** Faithfulness **3.80**, Relevance **3.80**, Recall **5.00**, Completeness **3.90**
- **Variant (hybrid_rrf):** Faithfulness **3.80**, Relevance **3.80**, Recall **5.00**, Completeness **3.60**

=> **Hybrid không cải thiện overall**, dù vẫn giữ Recall cao.

### 2.2 “Noise Overload” là failure mode chính của Hybrid
Quan sát chung khi bật BM25 + fusion:
- BM25 kéo về nhiều chunk “đúng keyword nhưng lệch ngữ cảnh”, làm context block bị loãng.
- LLM dễ **mất trọng tâm** khi tổng hợp, dẫn đến giảm `Completeness` (thiếu bước/quy trình) dù `Recall` vẫn cao.

### 2.3 Abstain/Insufficient-context phải cực kỳ chặt
Một điểm nhóm rút ra từ kết quả variant là: khi query thiếu dữ kiện hoặc docs không có câu trả lời, **format abstain và cách diễn đạt** ảnh hưởng trực tiếp đến điểm `Faithfulness/Relevance`.  
Baseline xử lý “an toàn” hơn; variant cần tinh chỉnh prompt/guardrails để tránh trả lời quá ngắn hoặc không đúng kỳ vọng rubric.

---

## 3) Quyết định cuối cùng và kế hoạch tối ưu tiếp theo

### 3.1 Quyết định cuối cùng (để ưu tiên điểm)
- Dùng **Dense baseline** làm cấu hình chính khi cần kết quả ổn định.
- Giữ **Hybrid + RRF** như cấu hình thử nghiệm; chỉ bật khi chắc chắn có biện pháp giảm nhiễu.

### 3.2 Nếu có thêm thời gian
Nhóm ưu tiên 2 hướng:
1) **Reranking (cross-encoder / rerank model)** sau retrieval để lọc nhiễu trước khi build context.  
2) **Metadata-driven filtering** (theo `department`, `effective_date`, hoặc loại tài liệu) để giảm candidate set cho BM25/hybrid.

---
