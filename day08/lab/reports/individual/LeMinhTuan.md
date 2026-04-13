# Báo Cáo Cá Nhân — Lab Day 08: RAG Pipeline

**Họ và tên:** Lê Minh Tuấn  
**Vai trò trong nhóm:** Retrieval Owner  
**Ngày nộp:** 14/04/2026  
**Độ dài yêu cầu:** 500–800 từ

---

## 1. Tôi đã làm gì trong lab này? (100-150 từ)

> Mô tả cụ thể phần bạn đóng góp vào pipeline:
Với vai trò là **Retrieval Owner**, công việc chính của tôi tập trung ở **Sprint 1 (Indexing)** và **Sprint 3 (Tuning)**. 
- Ở Sprint 1, tôi trực tiếp viết logic cho script `index.py`. Thay vì dùng Fixed-size chunking cắt chữ mù quáng, tôi viết biểu thức chính quy (Regex) cắt tài liệu theo thẻ `=== Section ===` (Semantic Chunking). Tôi thiết lập `CHUNK_SIZE = 400` token và chừa vùng `OVERLAP = 80` đảm bảo thông tin mượt mà. Đồng thời trích xuất metadata (Source, Department, Date).
- Ở Sprint 3, tôi implement thuật toán tìm kiếm **Hybrid Search** trong `rag_answer.py`. Tôi viết bộ từ khoá cài đặt thuật toán lấy top điểm BM25 (Sparse) gộp chung với vector (Dense), dùng cơ chế **Reciprocal Rank Fusion (RRF)** dung hoà kết quả. Nhờ bộ lõi "database sạch" và "retrieval mượt" tôi tạo ra, các bạn phụ trách (Tech Lead/Eval) ở Sprint 2 và 4 có context chuẩn để nhét vào Generated Prompt.

---

## 2. Điều tôi hiểu rõ hơn sau lab này (100-150 từ)

> Chọn 1-2 concept từ bài học mà bạn thực sự hiểu rõ hơn sau khi làm lab.
Sau khi đụng tay vào code, tôi sáng tỏ hai khái niệm: **Reciprocal Rank Fusion (RRF)** và **Distance vs Similarity**.
Thứ nhất, trên lớp tôi nghĩ Dense (vector ngữ nghĩa) là bá đạo nhất, nhưng thực hành cho thấy với dữ liệu chứa nhiều quy trình / mã lỗi (Ví dụ: "SLA ticket P1", "ERR"), cơ chế Dense thường bị "trôi" xa do từ khoá ngắn không mang nặng yếu tố ngữ nghĩa. Việc kết hợp BM25 (Sparse) giúp tóm chặt các từ khoá cứng, và RRF sẽ hoà quyện điểm hạng của 2 bên làm một, tối đa được thế mạnh bù trừ.
Thứ hai, tôi hiểu rõ cơ chế tính điểm của khoảng cách vector qua câu lệnh `1 - results["distances"][0][i]`. Máy tính đo độ vênh (Khoảng cách Cosine - càng gần 0 càng tốt), còn con người cần Độ tương đồng Cosine (càng gần 1 càng tốt).

---

## 3. Điều tôi ngạc nhiên hoặc gặp khó khăn (100-150 từ)

> Điều gì xảy ra không đúng kỳ vọng?
Điều ngạc nhiên lớn là khi tôi làm thuật toán **MMR (Maximal Marginal Relevance)** đa dạng hoá, giả thuyết ban đầu của tôi là MMR bao giờ cũng tốt hơn việc lấy top N ngớ ngẩn (Simple Truncate). 
Tuy nhiên thực tế, lúc đầu chỉnh tham số $\lambda$ (lambda parameter) sai lệch, MMR quá ưu tiên sự "khác biệt" của đoạn văn, dẫn tới việc vứt bỏ một đoạn văn RẤT GẦN ý nghĩa câu hỏi mà lại lôi vào một chunk rác không liên quan chỉ vì nó "đa dạng". Lỗi này tốn kha khá thời gian debug ở bước Generation, khi tôi ngớ người vì context block sinh ra không đáp ứng đủ bằng chứng. Quá trình chọn điểm cân bằng MMR tốn sức hơn tôi nghĩ.

---

## 4. Phân tích một câu hỏi trong scorecard (150-200 từ)

> Chọn 1 câu hỏi trong test_questions.json mà nhóm bạn thấy thú vị.

**Câu hỏi:** "Mức phạt vi phạm SLA P1 là bao nhiêu?" (Nhóm câu hỏi đánh lừa về Abstain/Anti-hallucination)

**Phân tích:**
- **Baseline (Chỉ Dense):** Lúc chạy thử baseline, kết quả có hiện tượng bị chập chờn. Có lúc LLM trả về đúng là "Không có thông tin trong tài liệu", có lúc LLM lại bị ảo giác (hallucinate) tự bịa một mức phạt chung chung do trong top 3 context có một mẩu chữ rác mang ý nghĩa na ná về quy trình phạt, làm nhiễu Generation. => Điểm: Kém.
- **Lỗi nằm ở đâu?** Lỗi nằm chéo ở hai khâu: Lỗi *Retrieval* vì Dense đã đưa lố một chunk "hơi gần nghĩa nhưng không trúng đích" vào làm thông tin nhiễu; lỗi ở *Generation Prompt* chưa đủ "lạnh" để ép cấm tuyệt đối suy diễn.
- **Variant có cải thiện không? Tại sao?** Rõ ràng! Khi áp dụng Hybrid + MMR, thông tin đưa lên được chắt lọc chuẩn hơn. Không có từ khoá "mức phạt vi phạm" khớp trực tiếp bằng BM25, dẫn đến điểm RRF của chunk nhiễu kia bị giáng xuống, nhường chỗ cho các đoạn văn không gợi bẫy. Kèm theo prompt đã tune, hệ thống dứt khoát Abstain: "Tôi xin lỗi, thông tin này không có trong tài liệu" lấy trọn điểm Penalty Rule.

---

## 5. Nếu có thêm thời gian, tôi sẽ làm gì? (50-100 từ)

> 1-2 cải tiến cụ thể bạn muốn thử.
Dựa trên Eval Scorecard, tôi sẽ đào sâu hai thứ:
1. **Tinh chỉnh Cross-Encoder (Reranker):** Do giới hạn resource ở Sprint 3 nên nhóm mới dừng ở Hybrid RRF, tôi muốn nhét thử mô hình `bge-reranker` ở bước cuối nhằm tính được trọng số Relevance điểm chữ qua chữ mạnh mẽ hơn rẽ ngang RRF.
2. **Dynamic Chunk Sizing:** Thử nghiệm cắt size động (Dynamic chunk) thay vì "căng" 400 token, vì các tài liệu Policy thường kéo dài liên kết luận điểm sang tận phiên đoạn sau, 400 token đôi khi bẻ gãy một mệnh đề nếu tác giả viết đoạn văn quá dài.
