# Scorecard: baseline_dense
Generated: 2026-04-13 14:41

## Summary

| Metric | Average Score |
|--------|--------------|
| Faithfulness | 3.80/5 |
| Relevance | 3.80/5 |
| Context Recall | 5.00/5 |
| Completeness | 3.90/5 |

## Per-Question Results

| ID | Category | Faithful | Relevant | Recall | Complete | Notes |
|----|----------|----------|----------|--------|----------|-------|
| q01 | SLA | 5 | 5 | 5 | 5 | Mọi thông tin trong Answer đều chính xác và có thể tìm thấy trong Context, cụ thể là thời gian SLA cho ticket P1 và thời gian phản hồi ban đầu. |
| q02 | Refund | 5 | 5 | 5 | 5 | Mọi thông tin trong Answer đều đúng và có thể tìm thấy trong Context. Cụ thể, thông tin về việc khách hàng có thể yêu cầu hoàn tiền trong vòng 7 ngày làm việc từ thời điểm xác nhận đơn hàng được nhắc đến rõ ràng trong Chunk 1 và Chunk 2. |
| q03 | Access Control | 5 | 5 | 5 | 5 | Thông tin trong Answer hoàn toàn chính xác. Yêu cầu cấp quyền Level 3 cần được phê duyệt bởi Line Manager, IT Admin và IT Security như đã nêu trong Chunk 1 của Context. |
| q04 | Refund | 5 | 5 | 5 | 5 | Thông tin trong Answer hoàn toàn phù hợp với Context, cụ thể là thông tin về sự không hoàn tiền đối với sản phẩm kỹ thuật số như license key và subscription đã được nêu rõ trong Chunk 1. |
| q05 | IT Helpdesk | 5 | 5 | 5 | 5 | Mọi thông tin trong Answer đều có thể tìm thấy trong Context, cụ thể là thông tin về việc tài khoản bị khóa sau 5 lần đăng nhập sai và cách mở khóa. |
| q06 | SLA | 5 | 5 | 5 | 5 | Tất cả thông tin trong Answer đều có thể tìm thấy trong Context và không có thông tin bịa đặt. Quy trình escalation cho sự cố P1 được mô tả chi tiết và đúng với nội dung trong các Chunk. |
| q07 | Access Control | 1 | 1 | 5 | 1 | Answer sai lệch vì nó khẳng định thông tin không có trong tài liệu nội bộ, trong khi thực tế Context cung cấp các thông tin chi tiết về quy trình cấp phép truy cập. |
| q08 | HR Policy | 5 | 5 | 5 | 5 | Mọi thông tin trong Answer đều được xác nhận từ Context, cụ thể là các điều kiện làm remote và yêu cầu phê duyệt của Team Lead. |
| q09 | Insufficient Context | 1 | 1 | None | 2 | Answer không có thông tin nào liên quan đến nội dung trong context và tuyên bố rằng thông tin không có trong tài liệu nội bộ là không chính xác, vì có thông tin rõ ràng về quy trình và chính sách liên quan đến access request. |
| q10 | Refund | 1 | 1 | 5 | 1 | Answer không cung cấp thông tin nào từ Context và khẳng định rằng thông tin không có trong tài liệu nội bộ, điều này không phản ánh chính xác các quy trình hoàn tiền và điều kiện yêu cầu mà nằm trong Context. |
