# Nhóm thực hiện K4 Track 3B

Tên nhóm: CaoDucAnh-Eval-Guard

Repo chung: https://github.com/CDanh0301/K4-L3-DAY24-CaoDucAnh-Eval-Guard

Trưởng nhóm/người nộp: Cao Đức Anh - 2A202602754

Ngày lab/deadline áp dụng: 11:59 AM (13/10/2026) — Asia/Ho_Chi_Minh (UTC+07:00)

Commit dùng để chấm: 140a6ac6021592f8e05c1c1a09c2da23928f5c5d

Mỗi nhóm nộp một repo. Điền đủ thành viên và một mục INDIVIDUAL cho từng người.
Thêm/xóa rows để khớp nhóm thật; mẫu ba rows không ép nhóm phải có ba người.

## Thành viên và phân công

| STT | Họ và tên | MSSV | Vai trò / công việc thật | Bằng chứng |
|---|---|---|---|---|
| 1 | Cao Đức Anh | 2A202602754 | Tích hợp toàn bộ Phase A, B, C và chạy thực nghiệm | PR/commit 140a6ac6021592f8e05c1c1a09c2da23928f5c5d; artifacts/ragas_50q.json, judge_results.json, guard_results.json |

Người tích hợp/chạy batch/nộp link: Cao Đức Anh, hoàn thành toàn bộ thí nghiệm thực tế và báo cáo, commit 140a6ac6021592f8e05c1c1a09c2da23928f5c5d.

## Môi trường và chuẩn bị trước lớp

OS/Python/compiler: Windows 11 64-bit, Python 3.11.9, MSVC v143 compiler cho Annoy.
Virtualenv/dependencies/imports: .venv cài đặt đầy đủ từ requirements/requirements.txt và constraints-core.txt, pyarrow 16.1.0, spaCy en_core_web_lg; lệnh kiểm tra import ragas, nemoguardrails, presidio_analyzer, spacy thành công.
Models/Qdrant/50 answers đã chuẩn bị: Qdrant v1.9.7 Docker container port 6333; embedding BAAI/bge-m3; reranker FlashRank; sinh đầy đủ 50 câu trả lời vào artifacts/answers_50q.json với generation_mode='llm', model gpt-4o-mini-2024-07-18.

## INDIVIDUAL

### CaoDucAnh-2A202602754

- Phần được giao và phần thực tế đã làm: Phụ trách toàn bộ bài lab bao gồm chuẩn bị pipeline RAG (m1-m5), chạy 50 answers, triển khai Phase A (RAGAS 4 metrics), Phase B (LLM Judge pairwise + calibration Cohen's kappa), Phase C (Presidio PII + NeMo Guardrails + đo latency P95) và hoàn thiện 3 báo cáo Markdown.
- Bằng chứng: artifacts/answers_50q.json, artifacts/ragas_50q.json, artifacts/judge_results.json, artifacts/guard_results.json, doc/reports/*.md.
- Cách kiểm tra: Đạt 86/86 offline pytest (test_phase_a, test_phase_b, test_phase_c, test_report_contracts, test_support). Chạy live experiment thành công trên toàn bộ 3 Phase và vượt qua check_lab.py.
- Điều đã học: Hiểu sâu về cách đánh giá toàn diện pipeline RAG, phát hiện bias vị trí trong LLM Judge thông qua kỹ thuật Swap-and-Average, tính chỉ số Cohen's kappa cho calibration nhãn người, thiết kế Guardrails nhiều tầng Presidio regex + NeMo semantic classification và đo lường độ trễ P95 tuần tự.
- Giới hạn/cách xử lý: NeMo Guardrails gọi API đám mây có độ trễ mạng cao (P95 đạt 639ms vượt budget 500ms). Hướng xử lý là đề xuất mô hình phân loại cục bộ trong kiến trúc CI/CD.
- Nguồn/AI hỗ trợ nếu có: Sử dụng AI Assistant để hỗ trợ rà soát mã nguồn và kiểm thử logic, toàn bộ kết quả thực nghiệm được chạy và kiểm chứng độc lập trên hệ thống máy thật.

## Cập nhật sau khi nộp

Bản nộp chính thức đúng hạn, không có cập nhật sau deadline.
