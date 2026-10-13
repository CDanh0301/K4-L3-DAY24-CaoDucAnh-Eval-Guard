# Phân tích lỗi Phase A

Nhóm: CaoDucAnh-Eval-Guard

Người phụ trách: Cao Đức Anh - 2A202602754

Nguồn: `artifacts/ragas_50q.json`; run timestamp: 2026-10-13T03:43:07.458517+00:00.
Generation mode/fallback và điều kiện chạy: Chạy pipeline RAG thực tế trên 50 câu với generation_mode='llm', model gpt-4o-mini, hybrid search Qdrant và cross-encoder reranker. Không có câu nào phải dùng extractive fallback.
Models generation/evaluator: generation: gpt-4o-mini-2024-07-18; evaluator: BAAI/bge-m3, gpt-4o-mini.

## Điểm theo distribution

| Metric | factual (20) | multi_hop (20) | adversarial (10) |
|---|---|---|---|
| faithfulness | 0.9083 | 0.4806 | 0.8500 |
| answer_relevancy | 0.7659 | 0.7278 | 0.7644 |
| context_precision | 0.9667 | 0.9458 | 0.9417 |
| context_recall | 0.8250 | 0.7958 | 0.6500 |
| avg_score | 0.8665 | 0.7375 | 0.8015 |

## Bottom 10 và nguyên nhân

| Rank | Question ID | Distribution | Question | Avg Score | Worst Metric | Diagnosis | Suggested Fix |
|---|---|---|---|---|---|---|---|
| 1 | 39 | multi_hop | So sánh yêu cầu mật khẩu giữa policy v1.0 và v2.0 về độ dài tối thiểu, thời hạn đổi và MFA. | 0.1250 | faithfulness | LLM hallucinating | Tighten system prompt, lower temperature |
| 2 | 7 | factual | Nghỉ phép không lương 20 ngày cần ai phê duyệt? | 0.3750 | faithfulness | LLM hallucinating | Tighten system prompt, lower temperature |
| 3 | 30 | multi_hop | So sánh quyền lợi bảo hiểm giữa nhân viên thử việc và nhân viên chính thức. | 0.3750 | faithfulness | LLM hallucinating | Tighten system prompt, lower temperature |
| 4 | 33 | multi_hop | Nhân viên Manager có thâm niên 12 năm: tổng phụ cấp hàng tháng và số ngày phép năm theo v2024 là bao nhiêu? | 0.3750 | faithfulness | LLM hallucinating | Tighten system prompt, lower temperature |
| 5 | 50 | adversarial | Nhân viên Manager có thể dùng VPN cá nhân khi WFH để tăng bảo mật thêm không? | 0.4167 | faithfulness | LLM hallucinating | Tighten system prompt, lower temperature |
| 6 | 5 | factual | Muốn mua thiết bị trị giá 55 triệu cần ai phê duyệt? | 0.5000 | answer_relevancy | Answer doesn't match question | Improve prompt template |
| 7 | 40 | multi_hop | Nhân viên cấp Manager có thời hạn thử việc 90 ngày, đang trong tháng thứ 3, phát hiện vi phạm bảo mật. Họ nên và không nên làm gì theo chính sách? | 0.7030 | faithfulness | LLM hallucinating | Tighten system prompt, lower temperature |
| 8 | 38 | multi_hop | Nhân viên đi công tác nước ngoài 4 ngày tại thành phố lớn, ở khách sạn 200 USD/đêm (3 đêm). Công ty thanh toán bao nhiêu tiền khách sạn? | 0.7039 | faithfulness | LLM hallucinating | Tighten system prompt, lower temperature |
| 9 | 31 | multi_hop | Nhân viên đi công tác trong nước, ở khách sạn 2 đêm với giá 1.500.000 VNĐ/đêm. Công ty thanh toán tối đa bao nhiêu cho tiền khách sạn? | 0.7075 | faithfulness | LLM hallucinating | Tighten system prompt, lower temperature |
| 10 | 44 | adversarial | Bao lâu phải đổi mật khẩu một lần? | 0.7108 | faithfulness | LLM hallucinating | Tighten system prompt, lower temperature |

Phân tích bằng chứng thực nghiệm:
- Câu 39 (Rank 1): Yêu cầu so sánh đa bước giữa hai phiên bản tài liệu mật khẩu v1.0 và v2.0. Reranker lấy được chunk của cả hai bản nhưng LLM bị lẫn lộn các thông số giữa phiên bản cũ và mới dẫn đến hallucination, faithfulness bị kéo xuống 0.
- Câu 7 (Rank 2): Nhân viên nghỉ không lương 20 ngày cần phê duyệt cấp cao; retrieval lấy trúng đoạn nghỉ phép thông thường nhưng thiếu chunk điều khoản nghỉ dài hạn trên 14 ngày.
- Câu 50 (Rank 5): Câu hỏi bẫy về VPN cá nhân. Reference cấm hoàn toàn VPN bên ngoài, nhưng LLM trả lời mơ hồ hoặc suy diễn rằng có thể dùng nếu an toàn, làm giảm tính trung thực so với policy gốc.

## Matrix và nhóm yếu nhất

| Metric | factual | multi_hop | adversarial |
|---|---|---|---|
| faithfulness | 3 | 15 | 2 |
| answer_relevancy | 11 | 2 | 1 |
| context_precision | 2 | 0 | 1 |
| context_recall | 4 | 3 | 6 |

Nhóm yếu theo mean thấp nhất: multi_hop; mean: 0.7375.
Worst metric xuất hiện nhiều nhất: faithfulness; số câu: 20/50 câu.

Nguyên nhân nhóm multi_hop yếu nhất:
Tập câu hỏi multi_hop đòi hỏi tổng hợp thông tin từ nhiều tài liệu hoặc nhiều phần khác nhau trong cùng chính sách ví dụ: vừa tính phụ cấp vừa tính phép năm theo thâm niên, hoặc so sánh chính sách qua các năm. Khi retrieval chỉ lấy top 3 chunks, một số chunk cần thiết bị bỏ sót hoặc thông tin bị phân mảnh khiến LLM phải tự suy đoán thông tin thiếu, dẫn đến tỷ lệ faithfulness thấp (chỉ 0.4806) và chiếm tới 15/20 ca lỗi faithfulness.
Đề xuất cải tiến:
Tăng số lượng retrieval chunks trước khi rerank (BM25_TOP_K và DENSE_TOP_K từ 20 lên 30), điều chỉnh RERANK_TOP_K từ 3 lên 5 để bao quát đủ ngữ cảnh liên kết. Thắt chặt system prompt yêu cầu từ chối trả lời nếu ngữ cảnh không đủ thông tin thay vì tự suy diễn.

## Bẫy adversarial và giới hạn

So sánh giữa các nhóm:
- Điểm trung bình của nhóm factual đạt cao nhất (0.8665), trong khi adversarial đạt 0.8015 và multi_hop đạt 0.7375.
- Trong nhóm adversarial (10 câu), context_recall đạt mức thấp nhất (0.6500) do các câu hỏi này cố tình cài cắm giả định sai hoặc hỏi về các quyền lợi không tồn tại ví dụ: dùng NordVPN, hỏi chính sách v1.0 đã hết hiệu lực. Điểm số thấp ở đây phản ánh đúng tính chất bẫy thông tin và xung đột phiên bản.
- Giới hạn: Tập mẫu 50 câu là tập kiểm thử nhỏ; model đánh giá gpt-4o-mini có thể có độ trễ phán đoán với ngữ cảnh tiếng Việt phức tạp.
