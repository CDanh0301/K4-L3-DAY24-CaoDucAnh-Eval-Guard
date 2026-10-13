# Đánh giá LLM Judge Phase B

Nhóm: CaoDucAnh-Eval-Guard

Người phụ trách: Cao Đức Anh - 2A202602754

Nguồn: `artifacts/judge_results.json`; run timestamp: 2026-10-13T03:43:48.638923+00:00.
Models generation/judge: generation: gpt-4o-mini-2024-07-18; judge: gpt-4o-mini.

## Năm cặp A/B và hai lượt

Nguồn cố định: A từ artifacts/answers_50q.json:answer; B từ
fixtures/human_labels_10q.json:model_answer; reference từ fixtures/test_set_50q.json:ground_truth.
Ids: 1, 5, 12, 21, 41. Không đổi nguồn sau khi xem kết quả judge.

| ID | Candidate A | Candidate B | Pass 1 Winner | Pass 2 Winner (quy về A/B) | Final Winner | Position Consistent | Scores Pass 1 (A/B) | Scores Pass 2 (A/B) |
|---|---|---|---|---|---|---|---|---|
| 1 | Nhân viên được nghỉ 3 ngày làm việc khi kết hôn. | Nhân viên được nghỉ 3 ngày làm việc có lương khi kết hôn. | B | B | B | True | 0.8 / 1.0 | 0.8 / 1.0 |
| 5 | Cần thêm phê duyệt của Kế toán trưởng. | Cần Giám đốc phòng ban phê duyệt. | B | tie | tie | False | 0.2 / 0.8 | 0.0 / 0.0 |
| 12 | Thưởng Tết tối thiểu cho nhân viên chính thức có từ 6 tháng trở lên là 1 tháng lương. | Nhân viên được thưởng Tết tối thiểu 1 tháng lương. | A | tie | tie | False | 1.0 / 0.7 | 1.0 / 1.0 |
| 21 | Nhân viên 9 năm thâm niên được 18 ngày phép (15 + 3). Lương của nhân viên này nằm trong khoảng của nhân viên chính thức, hưởng 15 ngày phép năm có lương. | Được nghỉ 18 ngày phép (15 cơ bản + 3 thâm niên). Lương Senior từ 20 đến 35 triệu. | B | B | B | True | 0.7 / 0.9 | 0.6 / 0.9 |
| 41 | Nhân viên không được thanh toán tiền làm thêm giờ vào ngày nghỉ tuần nếu không có phê duyệt trước bằng văn bản của Quản lý trực tiếp. | Không, nhân viên không được tự ý OT. | A | A | A | True | 0.9 / 0.4 | 0.9 / 0.5 |

Position bias count/rate: 2 / 5 (40.0%).
Verbosity win rate: 0.667; decisive cases: 3 (trong đó A thắng và dài hơn: 1, B thắng và dài hơn: 1, tổng decisive: 3).

Giải thích:
- Ở 3 trường hợp nhất quán (ID 1, 21, 41), chất lượng giữa hai câu trả lời có sự cách biệt rõ ràng về tính chính xác và đầy đủ so với reference, nên LLM giữ nguyên quyết định bất kể thứ tự trình bày.
- Ở 2 trường hợp không nhất quán (ID 5, 12), chất lượng giữa hai câu trả lời tương đối suýt soát hoặc cả hai đều có khiếm khuyết nhỏ so với reference. Khi hoán đổi vị trí, LLM thay đổi phán đoán sang 'tie'. Kỹ thuật swap-and-average đã phát huy tác dụng tuyệt đối khi quy về 'tie' để tránh thiên vị vị trí.

## Calibration đúng model_answer

Helper đánh giá đúng question/model_answer đang có trong fixtures/human_labels_10q.json;
không lấy answer mới của pipeline. Prompt chỉ nhận question, answer, reference.
Không gửi human_label, human_note hoặc phân bố nhãn human vào prompt.

| Question ID | Model Answer Gốc | Nhãn Judge | Reasoning | Human Label | Agree |
|---|---|---|---|---|---|
| 1 | Nhân viên được nghỉ 3 ngày làm việc có lương khi kết hôn. | 1 | Câu trả lời chính xác, nêu đúng số ngày nghỉ và khẳng định có lương. | 1 | True |
| 5 | Cần Giám đốc phòng ban phê duyệt. | 0 | Câu trả lời thiếu phê duyệt của Tổng Giám đốc cho mức trên 50 triệu. | 0 | True |
| 12 | Nhân viên được thưởng Tết tối thiểu 1 tháng lương. | 1 | Đúng ý thiết yếu về mức thưởng Tết tối thiểu theo quy định công ty. | 1 | True |
| 21 | Được nghỉ 18 ngày phép (15 cơ bản + 3 thâm niên). Lương Senior từ 20 đến 35 triệu. | 1 | Chính xác cả về số ngày phép thâm niên và khung lương Senior. | 1 | True |
| 23 | Lương Lead từ 35-50 triệu, phụ cấp gửi xe 200k. | 1 | Đúng khung lương và mức phụ cấp theo tài liệu chính sách. | 1 | True |
| 29 | Được thanh toán 100% nếu có hóa đơn đỏ. | 0 | Sai điều kiện vì chính sách quy định mức trần thanh toán tối đa. | 0 | True |
| 33 | Manager thâm niên 12 năm: phụ cấp ăn trưa 1 triệu, phụ cấp điện thoại 1 triệu, tổng 2 triệu/tháng. Phép năm 17 ngày. | 1 | Tính toán chính xác các khoản phụ cấp và ngày phép theo thâm niên. | 1 | True |
| 41 | Không, nhân viên không được tự ý OT. | 0 | Trả lời cụt lủn, thiếu điều kiện có thể được duyệt nếu có văn bản trước. | 0 | True |
| 46 | Phải đổi mật khẩu mỗi 90 ngày. Mật khẩu cũ không được trùng 3 lần gần nhất. | 1 | Đúng quy chuẩn chính sách an toàn thông tin về mật khẩu v2.0. | 1 | True |
| 50 | Được, miễn là đảm bảo kết nối an toàn. | 0 | Câu trả lời sai hoàn toàn vì chính sách nghiêm cấm dùng VPN cá nhân. | 0 | True |

Cohen’s κ: 1.0 Hoàn hảo.
Observed agreement: 10/10 (100.0%). Expected agreement p_e = 0.50.
κ = (1.0 - 0.50) / (1.0 - 0.50) = 1.0.
Diễn giải: Mức độ đồng thuận giữa LLM Judge (gpt-4o-mini) và nhãn người đạt mức Almost Perfect tối đa, chứng minh prompt binary evaluation được thiết kế chặt chẽ, đánh giá đúng theo tiêu chí factual và điều kiện chính sách.

## Giới hạn và cách dùng judge

- Giới hạn: Kích thước mẫu calibration gồm 10 câu hỏi tiêu biểu, chưa đại diện cho toàn bộ các tình huống phức tạp ngoài thực tế.
- Tỷ lệ Position Bias đo được trên 5 cặp mẫu đạt 40.0%, cho thấy xu hướng LLM Judge bị ảnh hưởng bởi thứ tự xuất hiện của câu trả lời là có thật.
- Khuyến nghị triển khai: Trong môi trường production, bắt buộc áp dụng quy trình swap-and-average 2 lượt cho mọi so sánh pairwise, và chỉ tin cậy kết quả khi cả hai lượt đồng thuận. Kết hợp song song kiểm tra factual grounding với RAGAS faithfulness.
