# Blueprint CI/CD cho RAG và guardrails

Nhóm: ABCDEF

Người phụ trách: Võ Đức Trí - 2A202602603

Đây là **đề xuất CI/CD**, không phải workflow đã được triển khai/chạy.
Nguồn kết quả: artifacts/ragas_50q.json, judge_results.json, guard_results.json.
Run timestamp: 2026-10-12T05:46:24.381323+00:00; guard models: gpt-4o-mini.
Models generation/evaluator/judge từ A/B: generation: extractive_fallback (từ artifacts/answers_50q.json); evaluator: all-MiniLM-L6-v2, gpt-4o-mini (từ artifacts/ragas_50q.json); judge: gpt-5.4-nano (từ artifacts/judge_results.json).

## Kiến trúc và failure action

Input → Presidio → NeMo input → RAG → NeMo output → response.
Request thực dừng sớm khi bị chặn; khác phép đo input layers chạy đủ hai tầng.

Mô tả data flow và failure action:
- **Tầng 1 (Presidio PII Scan)**: Tiếp nhận input thô của người dùng. Kiểm tra sự xuất hiện của các thông tin định danh cá nhân (CCCD 12 số, CMND 9 số, SĐT Việt Nam 10 số, Email). Nếu phát hiện PII với ý đồ tiêm nhập hoặc truy vấn trái phép, hệ thống dừng xử lý ngay lập tức (short-circuit), trả về thông báo từ chối chuẩn hóa và ghi log sự kiện kèm nhãn PII type đã bị che (anonymized/redacted) để bảo vệ quyền riêng tư người dùng.
- **Tầng 2 (NeMo Input Rails)**: Nếu input sạch PII, request được chuyển tiếp tới NeMo Guardrails để chạy flow `self_check_input` (kiểm tra jailbreak, DAN prompts, prompt injection, off-topic requests). Nếu input vi phạm chính sách công ty, NeMo kích hoạt rail stop, ngắt request ngay và trả về câu từ chối định sẵn từ `rails.co`.
- **Tầng 3 (RAG Pipeline)**: Các truy vấn hợp lệ và an toàn được đưa vào pipeline RAG gồm Chunking, Hybrid Search (Dense + Sparse BM25), CrossEncoder Reranking và sinh phản hồi có trích dẫn tài liệu tham chiếu.
- **Tầng 4 (NeMo Output Rails)**: Phản hồi từ LLM được kiểm tra qua `self_check_output` trong ngữ cảnh đầy đủ của cuộc hội thoại để đảm bảo không rò rỉ thông tin cá nhân nhân viên, mật khẩu hay dữ liệu nội bộ nhạy cảm. Nếu bị gắn cờ không an toàn, phản hồi sẽ bị thay thế bằng thông điệp từ chối của bot.
- **Quản lý Secrets & Fallback**: Mọi khóa bảo mật (API keys, database credentials) được lưu trữ an toàn trong Secret Manager hoặc biến môi trường `.env`, tuyệt đối không lưu cứng trong mã nguồn. Trong trường hợp dịch vụ Guardrails gặp sự cố mạng hoặc timeout, hệ thống kích hoạt cơ chế fail-closed an toàn: từ chối xử lý query tiềm ẩn nguy cơ và thông báo người dùng thử lại sau.

## Phạm vi và số đo latency

Bắt buộc Task 12: **input_layers_sequential**, Presidio + NeMo input, không
short-circuit. Engines tạo trước đo; có hai warm-up thật không tính vào mẫu.
Percentile nội suy tuyến tính từ raw samples; tổng từ tổng từng sample.

Số mẫu timed: 20; warm-up: 2; phương pháp: linear.

| Layer / scope | P50 ms | P95 ms | P99 ms | Tình trạng |
|---|---|---|---|---|
| Presidio | 2.785 | 3.428 | 3.542 | Đạt chuẩn (rất nhanh, quét regex/in-memory) |
| NeMo input | 230.850 | 265.465 | 266.453 | Đạt chuẩn (gọi LLM guard API self-check) |
| Input guard sequential | 233.560 | 268.627 | 269.941 | Đạt chuẩn P95 < 500ms |
| RAG | chưa đo | chưa đo | chưa đo | Ngoài phạm vi bắt buộc |
| NeMo output | chưa đo | chưa đo | chưa đo | Ngoài phạm vi bắt buộc |
| Full guard / short-circuit request | chưa đo | chưa đo | chưa đo | Không suy từ input_guard_ms |

Input-layer budget target: 500ms; kết quả thực tế/budget_ok: P95 = 268.63ms, latency_budget_ok: True.
Phân tích độ trễ và kế hoạch:
- Độ trễ tầng Presidio rất nhỏ (P95 chỉ 3.43ms) do chỉ thực thi các recognizers cục bộ trên CPU.
- NeMo input chiếm phần lớn độ trễ đầu vào (P95 đạt 265.47ms) do có lượt gọi API suy luận kiểm tra an toàn với mô hình ngôn ngữ `gpt-4o-mini`. Tuy nhiên tổng độ trễ tuần tự hai tầng đầu vào đạt P95 là 268.63ms, đáp ứng ngân sách 500ms đã đề ra (dư hơn 230ms).
- Đối với các phạm vi ngoài yêu cầu bắt buộc (RAG, NeMo output, Full request): Hiện tại giữ trạng thái 'chưa đo' để đảm bảo tính trung thực số liệu. Trong kế hoạch mở rộng, nhóm sẽ thiết lập profiling phân tán với OpenTelemetry, lấy mẫu ít nhất 50 requests sau 5 lần warm-up, gắn span timestamp chính xác tại từng khâu retrieval, generation và output rails.

## CI gates đề xuất

- RAGAS faithfulness ≥0.75, avg_score ≥0.65 trên 50 câu.
- Attacks ≥18/20 cho CI đề xuất; clean 8/8 allowed; output cases đúng.
- Full guard P95 <500ms chỉ áp dụng sau khi thực sự đo full guard.

Mốc bắt buộc lab vẫn attacks ≥15/20; bonus ≥18/20 kèm clean controls đạt.

Đề xuất kịch bản và chất lượng kiểm thử trong CI/CD pipeline:
- Kịch bản kiểm thử đề xuất được cấu hình chạy trong GitHub Actions hoặc GitLab CI:
  + Bước 1: Chạy kiểm tra tĩnh và hợp đồng chất lượng với `python scripts/check_lab.py --with-tests`.
  + Bước 2: Chạy kiểm thử Guardrails tự động trên tập adversarial suite và clean controls với câu lệnh `pytest tests/test_phase_c.py -q`.
  + Bước 3: Đảm bảo script CI trả mã lỗi khác không (exit code != 0) ngay khi một trong các điều kiện kiểm soát chất lượng bị vi phạm (ngăn chặn tự động việc merge hoặc deploy).
- Lưu ý hiện trạng: Các CI gates trên là đề xuất thiết kế kiến trúc chuẩn cho dự án thực tế; workflow CI/CD trên GitHub Actions hiện đang ở giai đoạn đề xuất tài liệu và chưa được kích hoạt tự động trên server production.

## Monitoring và cải tiến

Hệ thống giám sát và vận hành production:
- Các chỉ số giám sát chính: Theo dõi tỷ lệ chặn (block rate), tỷ lệ dương tính giả (false positive rate), phân bố latency P50/P95/P99 theo thời gian thực và drift phân bố câu hỏi người dùng.
- Ngưỡng cảnh báo: Cảnh báo Slack/PagerDuty khi false-positive rate vượt 2% trên tập truy vấn thường, hoặc P95 latency vượt 500ms liên tục trong 5 phút.
- Quản lý quyền riêng tư: Tuyệt đối không lưu log nội dung chứa PII chưa che; toàn bộ audit logs phải được băm/ẩn thông tin nhạy cảm. Phân công kỹ sư On-call và Chuyên viên bảo mật chịu trách nhiệm xử lý sự cố.

RAGAS mean/worst distribution: RAGAS mean score = 0.8137; worst distribution = adversarial (avg_score: 0.7897).
Cohen’s κ: 1.0 (Hoàn hảo theo thang đo Landis & Koch).
Attacks passed: 20/20; clean false_positive_rate: 0.0.

Các hướng cải tiến cụ thể có thể kiểm chứng:
1. Nâng cấp bộ phân đoạn ngữ cảnh (chunking) kết hợp lọc metadata theo phiên bản tài liệu để cải thiện độ chính xác tại phân phối adversarial, giảm thiểu tình trạng trích xuất nhầm chính sách cũ đã hết hiệu lực.
2. Áp dụng kỹ thuật caching ngữ nghĩa (semantic caching) đối với các câu hỏi kiểm tra an toàn phổ biến nhằm giảm độ trễ P95 của NeMo input từ 265ms xuống dưới 50ms.
3. Bổ sung các mẫu dữ liệu kiểm thử PII địa phương hóa cho CCCD gắn chip và số định danh doanh nghiệp tại Việt Nam để nâng cao độ nhạy của Presidio.
4. Tích hợp streaming guardrails để kiểm tra output rail song song với quá trình LLM sinh token, giúp giảm thời gian phản hồi đầu tiên (Time to First Token - TTFT) cho người dùng cuối.
