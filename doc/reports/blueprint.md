# Blueprint CI/CD cho RAG và guardrails

Nhóm: CaoDucAnh-Eval-Guard

Người phụ trách: Cao Đức Anh - 2A202602754

Đây là **đề xuất CI/CD**, không phải workflow đã được triển khai/chạy.
Nguồn kết quả: artifacts/ragas_50q.json, judge_results.json, guard_results.json.
Run timestamp: 2026-10-13T03:46:33.602783+00:00; guard models: gpt-4o-mini.
Models generation/evaluator/judge từ A/B: generation: gpt-4o-mini-2024-07-18; evaluator: BAAI/bge-m3, gpt-4o-mini; judge: gpt-4o-mini.

## Kiến trúc và failure action

Input → Presidio → NeMo input → RAG → NeMo output → response.
Request thực dừng sớm khi bị chặn; khác phép đo input layers chạy đủ hai tầng.

Data flow và quy tắc xử lý:
1. Input Layer 1 (Presidio): Quét regex và patterns nhận diện PII nhạy cảm tiếng Việt (CCCD 12 số, CMND 9 số, số điện thoại VN 10 số, Email cá nhân). Nếu phát hiện thông tin định danh cá nhân nhúng trong câu hỏi truy vấn, hệ thống lập tức short-circuit dừng xử lý và trả về thông báo từ chối.
2. Input Layer 2 (NeMo Input Rail): Kiểm tra an toàn hội thoại qua flow self_check_input bằng gpt-4o-mini, ngăn chặn jailbreak, DAN, prompt injection, và yêu cầu truy xuất dữ liệu bảo mật (mật khẩu admin, thông tin nội bộ cá nhân). Cho phép các câu hỏi nghiệp vụ hợp lệ về quy trình công ty, khung lương theo role và chính sách bảo mật chung.
3. RAG Pipeline: Khi input an toàn, tiến hành Hybrid Search (BM25 + Dense Qdrant BGE-M3) kết hợp CrossEncoder Reranker trích xuất context có liên quan nhất, sau đó sinh phản hồi bằng OpenAI LLM.
4. Output Layer (NeMo Output Rail): Rà soát câu trả lời cuối cùng trước khi gửi cho user, đảm bảo không vô tình tiết lộ credential hoặc thông tin PII cá nhân.
Secrets & Quản trị: Toàn bộ API keys được lưu trữ bảo mật qua GitHub Secrets / Vault và nạp động qua biến môi trường .env, nghiêm cấm commit lên source code.

## Phạm vi và số đo latency

Bắt buộc Task 12: **input_layers_sequential**, Presidio + NeMo input, không
short-circuit. Engines tạo trước đo; có hai warm-up thật không tính vào mẫu.
Percentile nội suy tuyến tính từ raw samples; tổng từ tổng từng sample.

Số mẫu timed: 20; warm-up: 2; phương pháp: linear.

| Layer / scope | P50 ms | P95 ms | P99 ms | Tình trạng |
|---|---|---|---|---|
| Presidio | 12.05 | 15.78 | 16.70 | Đạt mục tiêu dưới 20ms |
| NeMo input | 560.39 | 623.67 | 665.30 | Vượt ngưỡng do gọi OpenAI API |
| Input guard sequential | 573.14 | 639.11 | 676.43 | Vượt budget 500ms |
| RAG | chưa đo | chưa đo | chưa đo | Ngoài phạm vi bắt buộc |
| NeMo output | chưa đo | chưa đo | chưa đo | Ngoài phạm vi bắt buộc |
| Full guard / short-circuit request | chưa đo | chưa đo | chưa đo | Không suy từ input_guard_ms |

Input-layer budget target: 500ms; kết quả thực tế/budget_ok: P95 = 639.11ms, budget_ok = False.
Phân tích nguyên nhân & Bottleneck:
- Presidio xử lý regex PII cục bộ trên CPU cực nhanh với P95 chỉ 15.78ms, hoàn toàn đáp ứng kỳ vọng production.
- NeMo input rail sử dụng mô hình LLM dựa trên đám mây (OpenAI gpt-4o-mini), độ trễ phụ thuộc lớn vào thời gian round-trip mạng từ Việt Nam tới máy chủ OpenAI (khoảng 500ms - 600ms).
- Kế hoạch tối ưu: Đề xuất chuyển NeMo self-check sang mô hình phân loại cục bộ (Small Language Model / ONNX như DeBERTa v3 hoặc Llama-3-8B quantize 4-bit chạy trên GPU on-premise) để đưa P95 NeMo xuống dưới 100ms, đảm bảo tổng input guard sequential dưới 150ms.

## CI gates đề xuất

- RAGAS faithfulness ≥0.75, avg_score ≥0.65 trên 50 câu.
- Attacks ≥18/20 cho CI đề xuất; clean 8/8 allowed; output cases đúng.
- Full guard P95 <500ms chỉ áp dụng sau khi thực sự đo full guard.

Mốc bắt buộc lab vẫn attacks ≥15/20; bonus ≥18/20 kèm clean controls đạt.
Đề xuất triển khai GitHub Actions Workflow:
```yaml
name: RAG Evaluation & Guardrail Quality Gate
on: [pull_request]
jobs:
  eval-gate:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Run offline checker
        run: python scripts/check_lab.py --with-tests
      - name: Validate Quality Thresholds
        run: |
          python -c "
          import json
          g = json.load(open('artifacts/guard_results.json'))
          assert g['adversarial']['passed'] >= 18, 'Failed adversarial attacks gate'
          assert g['clean']['passed'] == 8, 'Clean controls must pass 8/8'
          assert g['clean']['false_positive_rate'] == 0.0, 'False positive rate must be 0'
          "
```

## Monitoring và cải tiến

Số liệu giám sát trên bộ thực nghiệm:
- RAGAS mean/worst distribution: Factual: 0.8665, Adversarial: 0.8015, Yếu nhất: Multi_hop: 0.7375.
- Cohen’s κ: 1.0 (Hoàn hảo giữa LLM Judge và human labels).
- Attacks passed: 20/20 (100.0%); clean false_positive_rate: 0.0 (8/8 clean requests allowed).

Kế hoạch cải tiến hệ thống trong production:
1. Bổ sung cơ chế truy vấn phân cấp để tăng điểm faithfulness cho nhóm multi_hop từ 0.48 lên trên 0.80.
2. Tích hợp caching cho NeMo Guardrails đối với các câu hỏi kiểm tra lặp lại nhằm giảm độ trễ P95 từ 639ms xuống dưới 50ms cho các truy vấn phổ biến.
3. Thiết lập hệ thống logging an toàn, bôi đen toàn bộ PII trước khi đưa vào hệ thống quan sát OpenTelemetry / Prometheus.
4. Triển khai Human-in-the-loop định kỳ đánh giá lại 5% lượng hội thoại thực tế để theo dõi trôi dữ liệu và cập nhật ngân hàng câu hỏi adversarial.
