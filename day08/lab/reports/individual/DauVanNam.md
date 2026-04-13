# Báo Cáo Cá Nhân — Lab Day 08: RAG Pipeline

**Họ và tên:** Đậu Văn Nam
**Vai trò:** Tech Lead
**Ngày nộp:** 13/04/2026

---

## 1. Mình đã làm gì trong buổi lab này?

Với vai trò là Tech Lead của dự án, mình chịu trách nhiệm bao quát toàn bộ quy trình từ khâu thiết kế kiến trúc hệ thống đến bước đánh giá cuối cùng. Cụ thể, mình đã trực tiếp tham gia:
- **Xây dựng Pipeline Indexing**: Phối hợp cùng nhóm để quyết định chiến thuật chunking theo đoạn văn và lựa chọn mô hình `text-embedding-3-small`.
- **Triển khai Logic Retrieval**: Cài đặt các hàm truy vấn lõi trong `rag_answer.py`, đảm bảo sự kết nối mượt mà giữa bộ lưu trữ ChromaDB và các mô hình ngôn ngữ.
- **Điều phối Evaluation**: Trực tiếp chạy và kiểm soát pipeline đánh giá 10 câu hỏi test, phân tích các chỉ số từ scorecard để đưa ra các nhận định về ưu nhược điểm của từng cấu hình (Baseline, Sparse, Hybrid).

---

## 2. Những điều mình nghiệm ra sau lab

Sau lab này, bài học lớn nhất đối với mình là sự phức tạp trong việc **cân bằng các metric**. Ví dụ, khi chúng mình chuyển sang dùng Sparse (Hybrid) trong Sprint 3, điểm **Faithfulness** (độ trung thực) tăng từ 3.80 lên 4.20, nhưng đổi lại điểm **Completeness** (độ đầy đủ) lại sụt giảm nghiêm trọng từ 3.90 xuống chỉ còn 2.90.

Điều này giúp mình hiểu rằng trong một hệ thống RAG thực tế, không có một cấu hình nào là "hoàn hảo" cho mọi tình huống. Với tư cách Tech Lead, mình nhận ra việc đọc hiểu sâu sắc các con số đánh giá quan trọng hơn nhiều so với việc chỉ tập trung vào việc viết code chạy được. Mỗi sự thay đổi nhỏ trong chiến thuật tìm kiếm đều dẫn đến những hệ quả khác nhau về chất lượng phản hồi của AI.

---

## 3. Khó khăn khi điều phối dự án

Khó khăn lớn nhất mình gặp phải là việc **phân tích nguyên nhân gốc rễ (Root Cause Analysis)** khi hệ thống hoạt động không đúng kỳ vọng. Đặc biệt là hiện tượng "Noise Overload" khi chạy Hybrid search. Việc nhìn thấy các tài liệu được retrieve về có Recall rất cao (5.00 tuyệt đối) nhưng câu trả lời sinh ra lại kém chất lượng làm mình khá đau đầu.

Qua việc soi lại từng context block, mình nhận ra rằng các đoạn văn bản mà BM25 kéo về dù chứa keyword chính xác nhưng đôi khi lại mang tính chất "nhiễu" cho LLM. Điều này đặt ra bài toán khó cho nhóm trong việc làm thế nào để "gạn đục khơi trong" trước khi đưa dữ liệu vào Generation phase.

---

## 4. Phân tích một case trong scorecard

**Câu hỏi:** "Quy trình escalation khi cần thay đổi quyền hệ thống khẩn cấp (SLA P1) như thế nào?" (q01)

**Phân tích dưới góc nhìn Tech Lead:**
Đây là một case study điển hình cho thấy sự khác biệt giữa các cấu hình:
- **Baseline (Dense)**: Hệ thống làm khá tốt phần ngữ nghĩa, bắt được ý của việc cấp quyền khẩn cấp.
- **Sparse (Sprint 3)**: Điểm Faithfulness tăng rõ rệt vì BM25 bắt được chính xác cụm "SLA P1" và "escalation" trong tài liệu. Tuy nhiên, do BM25 thường cắt vụn thông tin theo từ khóa, context mang về bị thiếu các bước thực hiện chi tiết dẫn đến điểm Completeness bị kéo xuống thấp.
- **Hybrid (Kế hoạch tiếp theo)**: Khi kết hợp lại, điểm Completeness đã hồi phục lên 3.60 nhưng vẫn chưa chạm tới mức 3.90 của Baseline.
- **Kết luận kỹ thuật**: Bài học rút ra là đối với các quy trình chính thống (SOP), Dense retrieval vẫn có ưu thế về việc bảo toàn tính mạch lạc của thông tin hơn là các phương pháp dựa trên từ khóa đơn thuần.

---

## 5. Nếu có thêm thời gian, mình sẽ thử gì tiếp theo?

Nếu có thêm thời gian, mình sẽ ưu tiên triển khai **Reranking** bằng mô hình Cross-Encoder để giải quyết triệt để vấn đề nhiễu thông tin. Kết quả từ thực nghiệm cho thấy Recall của chúng mình rất tốt, nhưng chất lượng sinh văn bản (Generation) bị ảnh hưởng bởi quá nhiều thông tin không liên quan. Một bước Rerank chuyên sâu sẽ giúp lọc lại top 3 đoạn văn thực sự đắt giá nhất, qua đó mình kỳ vọng sẽ đưa điểm Completeness lên trên mức 4.0.
