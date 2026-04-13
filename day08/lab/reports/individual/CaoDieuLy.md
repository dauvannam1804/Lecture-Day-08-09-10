# Báo Cáo Cá Nhân — Lab Day 08: RAG Pipeline

**Họ và tên:** Cao Diệu Ly  
**Vai trò trong nhóm:** Eval Owner  
**Ngày nộp:** 13/04/2026  

---

## 1. Tôi đã làm gì trong lab này?

Trong lab Day 08, tôi phụ trách vai trò Eval Owner, tập trung vào phần đánh giá chất lượng pipeline thay vì trực tiếp xây toàn bộ retrieval logic. Công việc chính của tôi là kiểm tra bộ `test_questions.json`, chạy scorecard cho baseline và các cấu hình variant, đọc kết quả theo từng metric, rồi dùng kết quả đó để quyết định cấu hình nào nên dùng khi chạy `grading_questions.json`. Sau khi có bộ câu hỏi grading, tôi chạy pipeline với nhiều cấu hình thử nghiệm, so sánh nhanh giữa `hybrid` và `dense top5`, sau đó chọn cấu hình `dense` với `top_k_select = 5` để tạo `logs/grading_run.json`. Tôi cũng kiểm tra log cuối cùng có đủ 10 câu, không có `PIPELINE_ERROR`, có timestamp trong khoảng 17:00-18:00, và mỗi dòng có đủ `id`, `question`, `answer`, `sources`, `chunks_retrieved`, `retrieval_mode`, `timestamp`.

## 2. Điều tôi hiểu rõ hơn sau lab này

Điều tôi hiểu rõ nhất là evaluation không chỉ là chạy một script để lấy điểm trung bình. Với RAG, một cấu hình có thể có Context Recall cao nhưng câu trả lời vẫn sai hoặc thiếu. Ví dụ, retriever có thể lấy đúng file nguồn, nhưng LLM vẫn abstain hoặc trả lời thiếu ý nếu context có nhiễu, câu hỏi nhiều điều kiện, hoặc chunk được chọn chưa đủ cụ thể. Tôi cũng thấy rõ sự khác nhau giữa faithfulness và completeness: một câu trả lời có thể không bịa, nhưng vẫn mất điểm vì thiếu chi tiết quan trọng. Vì vậy, khi đọc scorecard, tôi không chỉ nhìn average mà phải xem từng câu, nhất là các câu hard như cross-document, temporal reasoning và insufficient context.

## 3. Điều tôi gặp khó khăn hoặc thấy bất ngờ

Điều bất ngờ nhất là variant không phải lúc nào cũng tốt hơn baseline. Ban đầu tôi nghĩ hybrid retrieval sẽ phù hợp hơn vì bộ câu hỏi có nhiều keyword như SLA P1, VPN, Admin Access và các tên tài liệu. Nhưng khi chạy thử grading, cấu hình `hybrid top3` trả lời sai nghiêm trọng ở gq05: câu hỏi hỏi contractor có được cấp Admin Access không, nhưng pipeline lại abstain rằng không có thông tin trong tài liệu. Trong khi đó, `dense top5` lấy được ngữ cảnh tốt hơn và trả lời đúng rằng contractor có thể được cấp Admin Access, thời gian xử lý là 5 ngày làm việc và cần training security policy. Điều này cho thấy thêm BM25 không tự động làm pipeline tốt hơn; nếu context bị nhiễu hoặc top-k quá hẹp, generation vẫn có thể thất bại.

## 4. Phân tích một câu hỏi trong grading

**Câu hỏi:** gq05 — “Contractor từ bên ngoài công ty có thể được cấp quyền Admin Access không? Nếu có, cần bao nhiêu ngày và có yêu cầu đặc biệt gì?”

Đây là câu tôi dùng để quyết định cấu hình grading cuối cùng. Với `hybrid top3`, pipeline retrieve có liên quan đến `it/access-control-sop.md`, nhưng câu trả lời cuối lại là “thông tin này không có trong tài liệu nội bộ”. Failure mode chính không nằm ở indexing, vì tài liệu Access Control SOP đã được index và metadata vẫn có trong sources. Vấn đề nằm ở retrieval selection và generation: context đưa vào prompt chưa làm nổi bật đủ các chi tiết cần thiết cho một câu hỏi nhiều điều kiện, gồm phạm vi áp dụng cho contractor, Level 4 Admin Access, approver, thời gian 5 ngày và training bắt buộc. Khi chuyển sang `dense top5`, pipeline trả lời đúng phần lớn tiêu chí: contractor được cấp Admin Access, cần 5 ngày làm việc, và có yêu cầu training security policy, với source `it/access-control-sop.md`. Vì vậy tôi chọn `dense top5` để tạo `grading_run.json`, dù trước đó nhóm có thử nghiệm variant hybrid.

## 5. Nếu có thêm thời gian, tôi sẽ làm gì?

Tôi sẽ cải thiện evaluation theo hai hướng. Thứ nhất, thêm một bước rerank hoặc checklist theo grading criteria trước khi gọi LLM, đặc biệt cho các câu nhiều điều kiện như gq05 và gq06. Thứ hai, tôi sẽ lọc lại `sources` trong output để chỉ giữ các nguồn thực sự được cite trong answer, vì log hiện có lúc chứa thêm nguồn được retrieve nhưng không được dùng trực tiếp. Điều này sẽ giúp phần grading rõ hơn và giảm rủi ro bị đánh giá là citation không chính xác.
