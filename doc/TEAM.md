# Nhóm thực hiện K4 Track 3A

Tên nhóm: ABCDEF

Repo chung: https://github.com/hungdong19982003-sudo/K4-Track-3A-Day24-NguyenTheHung-2A202602381-Eval-Guard

Trưởng nhóm/người nộp: Nguyễn Thế Hưng - 2A202602381

Ngày lab/deadline áp dụng: 23:59 ngày 12/10/2026 — Asia/Ho_Chi_Minh (UTC+07:00)

Commit dùng để chấm: 7b3e3427f71b8ee66d3a4365df988e4e7e600868

Mỗi nhóm nộp một repo. Điền đủ thành viên và một mục INDIVIDUAL cho từng người.
Thêm/xóa rows để khớp nhóm thật; mẫu ba rows không ép nhóm phải có ba người.

## Thành viên và phân công

| STT | Họ và tên | MSSV | Vai trò / công việc thật | Bằng chứng |
|---|---|---|---|---|
| 1 | Nguyễn Thế Hưng | 2A202602381 | Trưởng nhóm, Lead Phase A: tích hợp m1-m5, sinh 50 answers, RagasEvaluation Tasks 1-4, failure clusters, tích hợp nghiệm thu repo | src/phase_a_ragas.py, scripts/setup_answers.py, artifacts/ragas_50q.json, doc/reports/failure_clusters.md |
| 2 | Lê Như Ý | 202602517 | Lead Phase B: Pairwise judge, swap-and-average khử position bias, Cohen kappa với human labels, bias report Tasks 5-8 | src/phase_b_judge.py, artifacts/judge_results.json, doc/reports/bias_report.md |
| 3 | Võ Đức Trí | 2A202602603 | Lead Phase C: Presidio PII, NeMo input/output rails, adversarial suite, đo P95 latency Tasks 9-12, đề xuất Blueprint | src/phase_c_guard.py, artifacts/guard_results.json, doc/reports/blueprint.md |

Người tích hợp/chạy batch/nộp link: Nguyễn Thế Hưng, phụ trách tổng hợp code, chạy toàn bộ test runner và check_lab.py, tạo commit và nộp link GitHub repo.

## Môi trường và chuẩn bị trước lớp

OS/Python/compiler: Windows 11, Python 3.11.9, MSVC compiler chưa tích hợp trong PATH cho gói Annoy.
Virtualenv/dependencies/imports: Khởi tạo .venv Python 3.11, cài đặt thành công test dependencies từ requirements-test.lock.txt, Presidio, spaCy, OpenAI SDK, Scipy; toàn bộ 86 unit tests offline đạt 100% PASS.
Models/Qdrant/50 answers đã chuẩn bị: Qdrant service đang chạy trên Docker cổng 6333; tải các model embedding BGE-M3 và CrossEncoder reranker từ module Day 18.

## INDIVIDUAL

Sao chép mẫu cho mỗi thành viên; tiêu đề họ tên không dấu-MSSV.

### NguyenTheHung-2A202602381

- Phần được giao và phần thực tế đã làm: Phụ trách Phase A và tích hợp pipeline RAG; hoàn thiện hàm phân nhóm distribution (Task 1), chạy RAGAS đánh giá 50 câu hỏi trên 4 metrics và giữ evaluator models (Task 2), trích xuất 10 câu yếu nhất kèm chẩn đoán và hướng khắc phục (Task 3), phân tích ma trận cụm lỗi và xác định nhóm yếu nhất theo điểm trung bình (Task 4) trong src/phase_a_ragas.py.
- Bằng chứng: Mã nguồn src/phase_a_ragas.py, artifacts/answers_50q.json, artifacts/ragas_50q.json, báo cáo doc/reports/failure_clusters.md và lịch sử commit git.
- Cách kiểm tra: Chạy unit test offline python -m pytest tests/test_phase_a.py -q đạt 6/6 PASS, kiểm tra tính toàn vẹn qua python scripts/check_lab.py.
- Điều đã học: Nắm vững bản chất 4 chỉ số RAGAS (faithfulness, answer_relevancy, context_precision, context_recall), phân biệt rõ lỗi tại khâu retrieval hay generation, và cách chẩn đoán lỗi trên các phân phối câu hỏi khác nhau.
- Giới hạn/cách xử lý: Câu hỏi adversarial dễ bị nhầm lẫn giữa các phiên bản chính sách; hướng xử lý là bổ sung lọc metadata thời gian và tăng cường reranker.
- Nguồn/AI hỗ trợ nếu có: Sử dụng AI hỗ trợ rà soát logic tính ma trận cụm lỗi, tự kiểm chứng bằng hợp đồng kiểm thử tại src/report_contracts.py.

### LeNhuY-202602517

- Phần được giao và phần thực tế đã làm: Phụ trách Phase B; hoàn thiện và thực thi toàn bộ pipeline LLM-as-Judge bao gồm hàm pairwise judge so sánh 2 câu trả lời A/B theo 3 tiêu chí (Task 5), cơ chế swap-and-average 2 lượt đảo vị trí để khử position bias (Task 6), tính toán hệ số Cohen's kappa đối chiếu 10 nhãn chuyên gia với độ đồng thuận đạt mức tuyệt đối κ = 1.0 (Task 7), xây dựng báo cáo kỹ thuật định lượng position bias (0.0%) và verbosity bias (Task 8) tại doc/reports/bias_report.md.
- Bằng chứng: Mã nguồn src/phase_b_judge.py, kết quả thực nghiệm artifacts/judge_results.json, báo cáo doc/reports/bias_report.md.
- Cách kiểm tra: Chạy unit test offline python -m pytest tests/test_phase_b.py -q đạt 8/8 PASS, chạy nghiệm thu qua python scripts/check_lab.py đạt PASS Phase B artifact và bias_report.md.
- Điều đã học: Nắm vững phương pháp thiết kế LLM-as-Judge chuẩn mực, cơ chế swap-and-average triệt tiêu Position Bias, cách tính và thang đo Landis-Koch cho hệ số Cohen's Kappa, cũng như phân biệt giữa thiên lệch độ dài giả tạo (verbosity bias) và câu trả lời dài hơn do chứa đầy đủ điều kiện nghiệp vụ.
- Giới hạn/cách xử lý: Số lượng mẫu kiểm chuẩn human labels (10 câu) còn nhỏ; hướng xử lý là duy trì cơ chế Human-in-the-Loop lấy mẫu kiểm toán 5-10% trong production CI/CD.
- Nguồn/AI hỗ trợ nếu có: Tham khảo lý thuyết đánh giá LLM-as-Judge và chuẩn hóa hợp đồng kiểm thử tại src/report_contracts.py.

### VoDucTri-2A202602603

- Phần được giao và phần thực tế đã làm: Phụ trách Phase C và Task 13; cài đặt module quét và ẩn PII với Presidio cho CCCD/CMND/SĐT VN/Email (Task 9a), thiết lập kiểm tra input rails (Task 9b) và output rails (Task 11) với NeMo Guardrails, chạy adversarial suite 28 inputs (Task 10), đo độ trễ P95 tuần tự input layers (Task 12) trong src/phase_c_guard.py và viết đề xuất doc/reports/blueprint.md.
- Bằng chứng: Mã nguồn src/phase_c_guard.py, dữ liệu artifacts/guard_results.json, báo cáo doc/reports/blueprint.md.
- Cách kiểm tra: Chạy unit test offline python -m pytest tests/test_phase_c.py -q đạt 12/12 PASS.
- Điều đã học: Hiểu cấu trúc kiến trúc phòng vệ an toàn nhiều lớp (Defense-in-Depth), vai trò thiết yếu của clean controls để đo lường tỷ lệ chặn nhầm (false-positive rate), và kỹ thuật đo độ trễ tuần tự có warm-up.
- Giới hạn/cách xử lý: Thư viện Annoy chưa biên dịch được trực tiếp trên Windows; xử lý bằng cách phân tách môi trường kiểm thử unit mock độc lập, bảo đảm an toàn logic và tính toán latency trung thực.
- Nguồn/AI hỗ trợ nếu có: Tham khảo tài liệu Presidio và NeMo Guardrails, đối chiếu các trường hợp kiểm thử với fixtures JSON.

## Cập nhật sau khi nộp

Bản nộp đầu tiên hoàn thiện đúng hạn quy định; không có cập nhật phát sinh sau deadline.
