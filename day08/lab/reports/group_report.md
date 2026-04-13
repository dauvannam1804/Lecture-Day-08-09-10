# Báo Cáo Nhóm — Lab Day 08: Full RAG Pipeline

**Tên nhóm:** Team RAG 
**Thành viên:**
| Tên | Vai trò | Email |
|-----|---------|-------|
| Đậu Văn Nam | Tech Lead | dauvannam321@gmail.com |
| Tuân | Retrieval Owner | tuan@example.com |
| Ly | Eval Owner | caodieuly1508@gmail.com |
| Nguyễn Trí Cao | Documentation Owner | tricao2003@gmail.com |

**Ngày nộp:** 13/04/2026  
**Repo:** `https://github.com/dauvannam1804/Lecture-Day-08-09-10`


---

## 1. Pipeline nhóm đã xây dựng (150–200 từ)

Hệ thống RAG của nhóm được thiết kế để xử lý truy vấn cho khối IT/CS Helpdesk, đòi hỏi phải cân bằng giữa khả năng hiểu ngữ nghĩa và độ chính xác của từ khóa chuyên ngành (mã lỗi, mã SLA).

**Chunking decision:**
Nhóm sử dụng cấu hình `chunk_size = 400 tokens` và `overlap = 80 tokens`. Phương pháp tách chunk được cấu hình theo **Paragraph-based splitting** (tách theo `\n\n`) thay vì cắt ký tự cứng. Điều này giúp giữ trọn vẹn ngữ nghĩa của từng điều khoản chính sách không bị đứt đoạn.

**Embedding model:**
Sử dụng mô hình OpenAI `text-embedding-3-small` để cân bằng giữa chi phí, tốc độ và độ phân giải không gian vector.

**Retrieval variant (Sprint 3):**
Nhóm đã phát triển và thử nghiệm các cấu hình **Sparse (BM25)** và **Hybrid (Dense + BM25 kết hợp qua RRF)**. Việc thêm Keyword Search (Sparse) nhằm khắc phục điểm yếu của Vector Similarity khi user truy vấn các từ khóa cực kỳ đặc thù như `ERR-403-AUTH` hoặc thẻ `SLA P1`.

---

## 2. Quyết định kỹ thuật quan trọng nhất (200–250 từ)

**Quyết định:** Chọn chiến lược Retrieval phù hợp để cải thiện điểm Completeness và Faithfulness, đánh đổi giữa Dense, Sparse, và Hybrid.

**Bối cảnh vấn đề:**
Nhóm nhận thấy với các tài liệu Helpdesk, user thường xuyên đưa các mã lỗi chính xác hoặc keyword viết tắt vào câu hỏi. Dense Retrieval thường mang lại ngữ cảnh mạch lạc nhưng đôi khi bỏ qua các văn bản chứa từ khóa chính xác nếu ý nghĩa tổng thể không "khớp".

**Các phương án đã cân nhắc:**

| Phương án | Ưu điểm | Nhược điểm |
|-----------|---------|-----------|
| **Dense Only** (Baseline) | Dễ triển khai, ngữ cảnh lấy về mạch lạc (High Completeness). | Yếu với Exact Match keyword. |
| **Sparse Only** (BM25) | Bắt keyword 100% chuẩn (cải thiện Faithfulness). | Bị đứt gãy ngữ cảnh khi lôi về các chunk chứa từ khóa chung chung, điểm Completeness nát. |
| **Hybrid (RRF)** | Cân bằng được cả hai. | Phức tạp, dễ dính "Noise Overload" (bị loãng context do mix nhiều nguồn). |

**Phương án đã chọn và lý do:**
Qua bàn bạc (Tech Lead + Retrieval Owner), nhóm quyết định triển khai **Hybrid (RRF)** nhưng cho chạy A/B Test cẩn thận từng biến. Ban đầu, kết quả chạy Sparse làm Faithfulness tăng nhưng Completeness giảm sâu (còn 2.90). Sau khi điều chỉnh sang Hybrid, điểm Completeness hồi phục lên 3.60. Đây là sự đánh đổi cần thiết để hệ thống bắt buộc phải hoạt động tốt trên các keyword mã lỗi, chấp nhận hi sinh một chút độ bao phủ câu từ.

**Bằng chứng từ scorecard/tuning-log:**
Theo `tuning-log.md`, khi dùng Baseline thì Faithfulness là 3.80, Completeness là 3.90. Khi chuyển sang Sparse, Faithfulness tăng vọt lên 4.20 nhưng Completeness tụt đáy xuống 2.90. 

---

## 3. Kết quả grading questions (100–150 từ)

**Ước tính điểm raw:** 92/98

**Câu tốt nhất:** Nhóm xử lý xuất sắc nhóm câu hỏi về thủ tục, ví dụ các câu liên quan đến quy trình "hoàn tiền" hoặc cấp quyền cơ bản. Lý do là Paragraph-based chunking đã phát huy tác dụng bảo toàn trọn vẹn danh sách các bước cần thực hiện, và Dense Retrieval làm tròn vai.

**Câu fail:** Câu hỏi q09 (ERR-403) hoặc q07 (dùng tên alias cũ là "Approval Matrix"). Root cause ở đây là do **Retrieval Noise**. Khi BM25 kéo về quá nhiều text chứa từ "Approval", nó làm loãng độ tập trung của prompt, LLM chỉ trả lời đúng câu chốt mà quên đi các phân tích phụ.

**Câu gq07 (abstain):** Pipeline đã chặn tốt lỗi Hallucination. Nếu không đủ Context, model kiên quyết trả lời "Tôi không biết" dựa vào dòng lệnh chặt chẽ trong System Prompt.

---

## 4. A/B Comparison — Baseline vs Variant (150–200 từ)

Nhóm đã sử dụng `tuning-log.md` để so sánh và lựa chọn hướng đi.

**Biến đã thay đổi (chỉ 1 biến):** Chiến thuật Retrieval (Dense vs Sparse)

| Metric | Baseline (Dense) | Variant (Sparse) | Delta |
|--------|---------|---------|-------|
| Faithfulness | 3.80 | 4.20 | +0.40 |
| Answer Relevance | 3.80 | 3.80 | 0.00 |
| Context Recall | 5.00 | 5.00 | 0.00 |
| Completeness | 3.90 | 2.90 | -1.00 |

**Kết luận:**
Việc sử dụng Sparse (chỉ dùng keyword) cho thấy ưu điểm về khả năng "bắt búng" từ khóa giúp câu trả lời trung thực (Faithfulness) hơn. Tuy nhiên, nó kéo theo hiện tượng **"Noise Overload"** khiến ngữ cảnh bị đứt đoạn, LLM thiếu đi các câu diễn giải nên điểm Completeness bị rớt thê thảm (-1.00). Chiến thuật cấp nhóm là dùng **Hybrid** để kéo lại điểm Completeness này.

---

## 5. Phân công và đánh giá nhóm (100–150 từ)

**Phân công thực tế:**

| Thành viên | Phần đã làm | Sprint |
|------------|-------------|--------|
| Nam | Thiết kế luồng kiến trúc (Pipeline) + Viết Logic chính. | 1, 2 |
| Tuân | Cài đặt ChromaDB, Embedding và thuật toán Sparse/Hybrid. | 1, 3 |
| Ly | Cài đặt logic Prompt LLM-as-Judge & Parse JSON Scorecard. | 4 |
| Cao | Viết Documents, vẽ luồng Mermaid, chạy A/B Test & Tuning. | All |

**Điều nhóm làm tốt:**
Chia task theo chuẩn luồng dữ liệu. Tài liệu hóa chặt chẽ ngay từ đầu (Tuning log) nên giúp tiết kiệm cực nhiều thời gian lúc ngồi debug.

**Điều nhóm làm chưa tốt:**
Cần đọc qua lab buổi học trước để lên lớp nắm bắt được luồng chạy nhanh hơn

---

## 6. Nếu có thêm 1 ngày, nhóm sẽ làm gì? (50–100 từ)

Với kết quả Context Recall đang ở mức 5.0 tuyệt đối nhưng Completeness lại trồi sụt do thông tin rác, nhóm chắc chắn sẽ tích hợp thêm **Reranker (Cross-Encoder)** vào Pipeline. Bước này sẽ đọc lướt lại Top 10 kết quả từ Hybrid, loại thẳng tay các kết quả chỉ trùng Keyword nhưng sai ý nghĩa, qua đó kỳ vọng Completeness sẽ vượt mốc 4.0.
