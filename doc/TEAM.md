# Nhóm thực hiện K4 Track 3A

Tên nhóm: Track3A-Nhom01

Repo chung: https://github.com/hungdong19982003-sudo/K4-Track-3A-Day24-NguyenTheHung-2A202602381-Eval-Guard

Trưởng nhóm/người nộp: Nguyễn Thế Hưng - 2A202602381

Ngày lab/deadline áp dụng: 23:59 ngày 12/10/2026 — Asia/Ho_Chi_Minh (UTC+07:00)

Commit dùng để chấm: af8d45084964177b90f4233777d5669b3df9856f

Mỗi nhóm nộp một repo. Điền đủ thành viên và một mục INDIVIDUAL cho từng người.
Thêm/xóa rows để khớp nhóm thật; mẫu ba rows không ép nhóm phải có ba người.

## Thành viên và phân công

| STT | Họ và tên | MSSV | Vai trò / công việc thật | Bằng chứng |
|---|---|---|---|---|
| 1 | Nguyễn Thế Hưng | 2A202602381 | Trưởng nhóm, Lead Phase A: tích hợp m1-m5, sinh 50 answers, RagasEvaluation Tasks 1-4, failure clusters, tích hợp nghiệm thu repo | src/phase_a_ragas.py, scripts/setup_answers.py, artifacts/ragas_50q.json, doc/reports/failure_clusters.md |
| 2 | Thành viên 2 | 2A202602382 | Lead Phase B: Pairwise judge, swap-and-average khử position bias, Cohen kappa với human labels, bias report Tasks 5-8 | src/phase_b_judge.py, artifacts/judge_results.json, doc/reports/bias_report.md |
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

### ThanhVien2-2A202602382

- Phần được giao và phần thực tế đã làm: Phụ trách Phase B; cài đặt hàm pairwise judge so sánh 2 câu trả lời A/B theo 3 tiêu chí (Task 5), triển khai swap-and-average đảo thứ tự để khử position bias (Task 6), tính hệ số Cohen kappa đối chiếu với 10 nhãn chuyên gia (Task 7), xây dựng báo cáo định lượng position bias và verbosity bias (Task 8) trong src/phase_b_judge.py.
- Bằng chứng: Mã nguồn src/phase_b_judge.py, dữ liệu artifacts/judge_results.json, báo cáo doc/reports/bias_report.md.
- Cách kiểm tra: Chạy unit test offline python -m pytest tests/test_phase_b.py -q đạt 8/8 PASS.
- Điều đã học: Hiểu sâu về các dạng thiên lệch (bias) thường gặp khi sử dụng LLM làm giám khảo, kỹ thuật swap-and-average giúp đảm bảo tính nhất quán, và thang đo Landis-Koch khi diễn giải hệ số Cohen kappa.
- Giới hạn/cách xử lý: Một số trường hợp câu trả lời dài hơn có xu hướng được ưu tiên; xử lý bằng prompt chuẩn hóa yêu cầu khắt khe về tính súc tích và căn cứ chính sách.
- Nguồn/AI hỗ trợ nếu có: Tham khảo lý thuyết về LLM evaluation bias, tự viết và kiểm thử lại logic tính Cohen kappa bằng tay.

### VoDucTri-2A202602603

- Phần được giao và phần thực tế đã làm: Phụ trách Phase C và Task 13; cài đặt module quét và ẩn PII với Presidio cho CCCD/CMND/SĐT VN/Email (Task 9a), thiết lập kiểm tra input rails (Task 9b) và output rails (Task 11) với NeMo Guardrails, chạy adversarial suite 28 inputs (Task 10), đo độ trễ P95 tuần tự input layers (Task 12) trong src/phase_c_guard.py và viết đề xuất doc/reports/blueprint.md.
- Bằng chứng: Mã nguồn src/phase_c_guard.py, dữ liệu artifacts/guard_results.json, báo cáo doc/reports/blueprint.md.
- Cách kiểm tra: Chạy unit test offline python -m pytest tests/test_phase_c.py -q đạt 12/12 PASS.
- Điều đã học: Hiểu cấu trúc kiến trúc phòng vệ an toàn nhiều lớp (Defense-in-Depth), vai trò thiết yếu của clean controls để đo lường tỷ lệ chặn nhầm (false-positive rate), và kỹ thuật đo độ trễ tuần tự có warm-up.
- Giới hạn/cách xử lý: Thư viện Annoy chưa biên dịch được trực tiếp trên Windows; xử lý bằng cách phân tách môi trường kiểm thử unit mock độc lập, bảo đảm an toàn logic và tính toán latency trung thực.
- Nguồn/AI hỗ trợ nếu có: Tham khảo tài liệu Presidio và NeMo Guardrails, đối chiếu các trường hợp kiểm thử với fixtures JSON.

## Cập nhật sau khi nộp

Bản nộp đầu tiên hoàn thiện đúng hạn quy định; không có cập nhật phát sinh sau deadline.
