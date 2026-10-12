# Phân tích lỗi Phase A

Nhóm: ABCDEF

Người phụ trách: Nguyễn Thế Hưng - 2A202602381

Nguồn: `artifacts/ragas_50q.json`; run timestamp: 2026-10-12T05:04:41.300072+00:00.
Generation mode/fallback và điều kiện chạy: extractive_fallback từ retrieval context top-1, thực thi trên Qdrant Docker v1.9.7 (localhost:6333) kết hợp BM25 Okapi tiếng Việt và Cross-Encoder reranker BAAI/bge-reranker-v2-m3.
Models generation/evaluator: Generation: [] (extractive_fallback), Evaluator: ['all-MiniLM-L6-v2', 'gpt-4o-mini'].

## Điểm theo distribution

| Metric | factual (20) | multi_hop (20) | adversarial (10) |
|---|---|---|---|
| faithfulness | 1.0000 | 1.0000 | 1.0000 |
| answer_relevancy | 0.8651 | 0.8911 | 0.9254 |
| context_precision | 0.7917 | 0.7417 | 0.7333 |
| context_recall | 0.7178 | 0.5507 | 0.5001 |
| avg_score | 0.8436 | 0.7958 | 0.7897 |

## Bottom 10 và nguyên nhân

| Rank | ID | Câu hỏi | Distribution | Avg Score | Worst Metric | Diagnosis | Suggested Fix |
|---|---|---|---|---|---|---|---|
| 1 | 18 | Cơ cấu điểm đánh giá hiệu suất gồm những thành phần nào và tỷ lệ ra sao? | factual | 0.4824 | answer_relevancy | Answer doesn't match question | Improve prompt template |
| 2 | 40 | Nhân viên cấp Manager có thời hạn thử việc 90 ngày, đang trong tháng thứ 3, phát hiện vi phạm bảo mật. Họ nên và không nên làm gì theo chính sách? | multi_hop | 0.4917 | context_precision | Too many irrelevant chunks | Add reranking or metadata filter |
| 3 | 38 | Nhân viên đi công tác nước ngoài 4 ngày tại thành phố lớn, ở khách sạn 200 USD/đêm (3 đêm). Công ty thanh toán bao nhiêu tiền khách sạn? | multi_hop | 0.4948 | context_precision | Too many irrelevant chunks | Add reranking or metadata filter |
| 4 | 3 | Phụ cấp ăn trưa hàng tháng là bao nhiêu? | factual | 0.5144 | context_precision | Too many irrelevant chunks | Add reranking or metadata filter |
| 5 | 41 | Nhân viên được nghỉ bao nhiêu ngày phép năm? | adversarial | 0.5441 | context_precision | Too many irrelevant chunks | Add reranking or metadata filter |
| 6 | 22 | Nếu cần mua một chiếc laptop 30 triệu cho nhân viên mới, ai phê duyệt và cần gì từ phòng CNTT? | multi_hop | 0.5687 | context_precision | Too many irrelevant chunks | Add reranking or metadata filter |
| 7 | 37 | Nhân viên tự ý xóa malware khỏi máy và chia sẻ thông tin sự cố này trên Slack nội bộ. Vi phạm những chính sách nào cụ thể? | multi_hop | 0.5740 | context_precision | Too many irrelevant chunks | Add reranking or metadata filter |
| 8 | 20 | Phụ cấp đi lại tối đa mỗi tháng là bao nhiêu và điều kiện để được hưởng là gì? | factual | 0.6304 | context_recall | Missing relevant chunks | Improve chunking or add BM25 |
| 9 | 44 | Bao lâu phải đổi mật khẩu một lần? | adversarial | 0.6490 | context_precision | Too many irrelevant chunks | Add reranking or metadata filter |
| 10 | 35 | Nhân viên Junior P1 có lương cơ bản 12 triệu vừa vào thử việc: nhận lương bao nhiêu và phụ cấp gì tháng đầu tiên? | multi_hop | 0.6551 | context_recall | Missing relevant chunks | Improve chunking or add BM25 |

Đối chiếu chi tiết bằng chứng từ answer, contexts và reference của các câu đại diện:
- Câu 18 (factual, ID 18): Câu hỏi yêu cầu cơ cấu thành phần điểm và tỷ lệ phần trăm đánh giá hiệu suất. Context trích xuất chứa mô tả chu kỳ đánh giá giữa kỳ và cuối năm mà thiếu đoạn quy định chi tiết tỷ lệ (60% KPI công việc, 20% năng lực cốt lõi, 20% tuân thủ kỷ luật). Do đó, câu trả lời trích xuất không đáp ứng trực diện câu hỏi, dẫn đến `answer_relevancy` là metric thấp nhất (0.4824 avg score).
- Câu 3 (factual, ID 3): Câu hỏi về phụ cấp ăn trưa hàng tháng (ground truth: 1.000.000 VNĐ/tháng). Quá trình hybrid search bị nhiễu do từ khóa "phụ cấp" xuất hiện rải rác trong nhiều văn bản khác (chính sách Mentor/Buddy, quy định mua sắm tài sản CNTT), kéo theo các chunk không chứa con số 1.000.000 VNĐ lên đầu danh sách retrieved contexts, làm `context_precision` và `context_recall` bị sụt giảm nghiêm trọng.
- Câu 40, 38, 22 (multi_hop): Yêu cầu kết hợp thông tin đa nguồn (Q40 kết hợp chính sách thử việc và quy trình xử lý sự cố bảo mật; Q38 kết hợp định mức công tác phí nước ngoài và hóa đơn khách sạn thực tế). Do top-k rerank bị giới hạn ở 3 chunks, pipeline chỉ truy xuất được thông tin của một trong hai tài liệu cần thiết, khiến chunk thứ hai bị bỏ sót, trực tiếp kéo tụt `context_precision` và `context_recall`.
- Câu 41, 44 (adversarial): Đây là các bẫy xung đột phiên bản tài liệu (version conflict). Tại câu 41, văn bản cũ `nghi_phep_nam_v2023.md` quy định 12 ngày phép, trong khi văn bản mới `nghi_phep_nam_v2024.md` nâng lên 14 ngày phép. Bộ tìm kiếm không có cơ chế lọc siêu dữ liệu ngày hiệu lực nên đã đưa chunk v2023 lên top, khiến câu trả lời bị xung đột chính sách. Tương tự, tại câu 44 về chu kỳ đổi mật khẩu, hệ thống bị xung đột giữa `mat_khau_v1.md` (90 ngày) và `mat_khau_v2.md` (180 ngày).

## Matrix và nhóm yếu nhất

| Metric | factual | multi_hop | adversarial | Total |
|---|---|---|---|---|
| faithfulness | 3 | 0 | 0 | 3 |
| answer_relevancy | 1 | 0 | 0 | 1 |
| context_precision | 3 | 5 | 3 | 11 |
| context_recall | 13 | 15 | 7 | 35 |
| Total | 20 | 20 | 10 | 50 |

Nhóm yếu theo mean thấp nhất: adversarial; mean: 0.7897.
Worst metric xuất hiện nhiều nhất: context_recall; số câu: 35.

Giải thích nguyên nhân nhóm yếu nhất và metric chủ đạo:
- Phân bố adversarial có điểm trung bình thấp nhất (0.7897) so với multi_hop (0.7958) và factual (0.8436) bởi vì nhóm này tập trung các câu hỏi bẫy về phiên bản văn bản hết hiệu lực (v2023 vs v2024), điều khoản phủ định hoặc chính sách không tồn tại. Bộ tìm kiếm HybridSearch và Cross-Encoder thuần semantic không có siêu dữ liệu về trạng thái hiệu lực nên dễ bị lừa bởi độ tương đồng ngữ nghĩa của tài liệu cũ.
- `context_recall` là điểm yếu lớn nhất (chiếm 35/50 câu có điểm worst metric). Nguyên nhân chính là do kích thước chunk nhỏ (256 tokens) khiến các điều kiện ràng buộc trong hợp đồng bị phân mảnh qua nhiều chunk, và giới hạn top-3 reranking chưa đủ dung lượng để gom toàn bộ các mảnh thông tin cần thiết.
- Đề xuất cải tiến cụ thể:
  1. Thêm bộ lọc metadata filter theo thời gian hiệu lực (`effective_date`) để chỉ tìm kiếm trên phiên bản văn bản đang áp dụng.
  2. Tăng số lượng context sau rerank từ top-3 lên top-5 cho nhóm câu hỏi phức hợp đa nguồn.
  3. Kiểm chứng hiệu quả bằng cách chạy lại toàn bộ pipeline đánh giá: `python scripts/setup_answers.py --live` và `python scripts/run_experiments.py --phase a --live`.

## Bẫy adversarial và giới hạn

So sánh với factual và multi_hop, nhóm adversarial thể hiện rõ rệt rủi ro khi hệ thống RAG vận hành trên kho tài liệu doanh nghiệp có tính cập nhật liên tục:
- Các trường hợp xung đột phiên bản (version conflict) như chính sách phép năm (câu 41) hay chính sách mật khẩu (câu 44) cho thấy việc tìm kiếm chỉ dựa vào độ tương đồng vector hoặc từ khóa BM25 là chưa đủ an toàn. Điểm số thấp hơn ở nhóm adversarial phản ánh việc hệ thống trích xuất nhầm tài liệu lỗi thời, chứ không chứng minh pipeline đã chủ động phát hiện và cảnh báo bẫy.
- Giới hạn thực nghiệm: Tập dữ liệu kiểm thử gồm 50 câu hỏi (20 factual, 20 multi_hop, 10 adversarial) đóng vai trò mẫu đo lường ban đầu. Lượt chạy hiện tại vận hành ở chế độ `extractive_fallback` dựa trên đoạn trích có độ liên quan cao nhất từ retriever thực tế. Trong môi trường production có LLM generative, cần kết hợp system prompt với các ràng buộc kiểm tra phiên bản nghiêm ngặt và guardrails để chủ động từ chối thông tin xung đột.
