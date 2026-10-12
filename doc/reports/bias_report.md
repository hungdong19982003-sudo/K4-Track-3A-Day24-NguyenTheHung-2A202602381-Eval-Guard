# Đánh giá LLM Judge Phase B

Nhóm: ABCDEF

Người phụ trách: Lê Như Ý - 202602517

Nguồn: `artifacts/judge_results.json`; run timestamp: 2026-10-12T05:31:08.962770+00:00.
Models generation/judge: generation: extractive_fallback; judge: gpt-5.4-nano.

## Năm cặp A/B và hai lượt

Nguồn cố định: A từ artifacts/answers_50q.json:answer; B từ fixtures/human_labels_10q.json:model_answer; reference từ fixtures/test_set_50q.json:ground_truth.
Ids: 1, 5, 12, 21, 41. Không đổi nguồn sau khi xem kết quả judge.

| ID | Câu hỏi | Candidate A | Candidate B | Winner Pass 1 | Winner Pass 2 | Final Winner | Consistent | Điểm Pass 1 (A/B) | Điểm Pass 2 (A/B) |
|---|---|---|---|---|---|---|---|---|---|
| 1 | Nhân viên được nghỉ bao nhiêu ngày khi kết hôn? | Nhân viên cần cung cấp giấy tờ chứng minh trong vòng 5 ngày làm việc... | Nhân viên được nghỉ 3 ngày làm việc có lương khi kết hôn. | B | B | B | True | 0.45 / 1.0 | 0.2 / 1.0 |
| 5 | Muốn mua thiết bị trị giá 55 triệu cần ai phê duyệt? | Mua sắm thiết bị CNTT cần có xác nhận của phòng CNTT... | Cần Giám đốc phòng ban phê duyệt. | tie | tie | tie | True | 0.1 / 0.1 | 0.05 / 0.05 |
| 12 | Thưởng Tết tối thiểu cho nhân viên chính thức có từ 6 tháng trở lên là bao nhiêu? | Chính sách thưởng Tết: Tất cả nhân viên chính thức từ 6 tháng trở lên được thưởng tối thiểu 1 tháng lương. | Nhân viên được thưởng Tết tối thiểu 1 tháng lương. | A | tie | tie | False | 1.0 / 0.8 | 0.98 / 1.0 |
| 21 | Một nhân viên Senior có 9 năm thâm niên được nghỉ bao nhiêu ngày phép năm và lương trong khoảng nào? | Ví dụ: nhân viên 9 năm thâm niên được 18 ngày phép (15 + 3)... | Được nghỉ 18 ngày phép (15 cơ bản + 3 thâm niên). Lương Senior từ 20 đến 35 triệu. | tie | B | tie | False | 0.88 / 0.95 | 0.75 / 0.95 |
| 41 | Nhân viên được nghỉ bao nhiêu ngày phép năm? | Trong thời gian nghỉ không lương, nhân viên không được hưởng lương... | Nhân viên được nghỉ 12 ngày phép năm. | B | B | B | True | 0.05 / 0.45 | 0.05 / 0.35 |

Position bias count/rate: 2 (40.0%).
Verbosity win rate: 0.0%; decisive cases: 2.

Phân tích định lượng và định tính:
- Về Position Bias: Tỷ lệ thiên vị vị trí là 2/5 cases (40.0%) tại câu hỏi 12 và câu hỏi 21. Cụ thể, khi đổi thứ tự truyền vào prompt giữa A và B, quyết định của LLM Judge có sự dao động (ở lượt 1 chọn A hoặc tie, ở lượt 2 lại chọn B hoặc tie). Điều này cho thấy LLM Judge chịu ảnh hưởng rõ rệt bởi vị trí xuất hiện của phương án. Nhờ áp dụng kỹ thuật swap-and-average, các trường hợp bất đồng giữa hai lượt đã được quy về 'tie' một cách khách quan, triệt tiêu hoàn toàn ưu thế của vị trí đứng trước.
- Về Verbosity Bias: Trong 2 decisive cases (câu 1 và câu 41, với người chiến thắng rõ ràng là B), tỷ lệ câu dài hơn thắng là 0/2 (0.0%). Answer B đều ngắn gọn hơn Answer A nhưng giành chiến thắng do Answer A là đoạn trích xuất extractive trích nhầm context (nhầm sang thủ tục giấy tờ hoặc nghỉ không lương), trong khi Answer B trả lời trực diện và chính xác hơn theo reference. Điều này chứng minh LLM Judge ưu tiên tính chính xác và bám sát reference hơn là độ dài văn bản.

## Calibration đúng model_answer

Helper đánh giá đúng question/model_answer đang có trong fixtures/human_labels_10q.json; không lấy answer mới của pipeline. Prompt chỉ nhận question, answer, reference. Không gửi human_label, human_note hoặc phân bố nhãn human vào prompt.

| ID | Câu hỏi | Model Answer Đánh Giá | Nhãn Judge | Reasoning của Judge | Nhãn Human | Agree |
|---|---|---|---|---|---|---|
| 1 | Nhân viên được nghỉ bao nhiêu ngày khi kết hôn? | Nhân viên được nghỉ 3 ngày làm việc có lương khi kết hôn. | 1 | Answer nêu đúng số ngày nghỉ (3 ngày làm việc có lương khi kết hôn) phù hợp với reference. Ý thiết yếu về số ngày và có lương là hoàn toàn chính xác. | 1 | True |
| 5 | Muốn mua thiết bị trị giá 55 triệu cần ai phê duyệt? | Cần Giám đốc phòng ban phê duyệt. | 0 | Reference quy định đơn hàng trên 50.000.000 VNĐ cần Tổng Giám đốc (CEO) phê duyệt, nhưng answer lại nói cần Giám đốc phòng ban phê duyệt nên không đúng thẩm quyền. | 0 | True |
| 12 | Thưởng Tết tối thiểu cho nhân viên chính thức có từ 6 tháng trở lên là bao nhiêu? | Nhân viên được thưởng Tết tối thiểu 1 tháng lương. | 1 | Câu trả lời nêu đúng mức thưởng Tết tối thiểu 1 tháng lương cho nhân viên chính thức có thời gian làm việc từ 6 tháng trở lên, phù hợp với reference. | 1 | True |
| 21 | Một nhân viên Senior có 9 năm thâm niên được nghỉ bao nhiêu ngày phép năm và lương trong khoảng nào? | Được nghỉ 18 ngày phép (15 cơ bản + 3 thâm niên). Lương Senior từ 20 đến 35 triệu. | 1 | Answer nêu đúng số ngày phép theo v2024 (15 cơ bản + 3 thâm niên = 18 ngày) và đúng khoảng lương Senior 20-35 triệu/tháng theo reference. | 1 | True |
| 23 | Nhân viên được tài trợ khóa học 25 triệu, nghỉ việc sau 8 tháng. Phải hoàn trả bao nhiêu? | Hoàn trả 25 triệu vì chưa đủ 1 năm cam kết. | 1 | Answer nêu đúng việc nghỉ sau 8 tháng (chưa đủ 1 năm) nên phải hoàn trả 100% chi phí khóa học 25.000.000 VNĐ, phù hợp reference. | 1 | True |
| 29 | Nhân viên tạm ứng 8 triệu, chưa thanh toán sau 30 ngày. Ai phê duyệt và phí phạt là bao nhiêu? | Trưởng phòng phê duyệt. Phạt 2% tháng trên 8 triệu. | 0 | Answer nêu đúng người phê duyệt nhưng sai phí phạt: reference tính quá hạn theo 15 ngày (2%/tháng × 8.000.000 × 15/30 = 80.000 VNĐ). Answer không quy đổi thời gian quá hạn. | 0 | True |
| 33 | Nhân viên Manager có thâm niên 12 năm: tổng phụ cấp hàng tháng và số ngày phép năm? | Phép: 19 ngày. Phụ cấp: 1.500.000 VNĐ/tháng. | 1 | Answer nêu đúng tổng phụ cấp 1.500.000 VNĐ/tháng và đúng số ngày phép năm 19 ngày (15 ngày cơ bản + 4 ngày thâm niên 12 chia 3 theo reference). | 1 | True |
| 41 | Nhân viên được nghỉ bao nhiêu ngày phép năm? | Nhân viên được nghỉ 12 ngày phép năm. | 0 | Câu trả lời nêu 12 ngày, nhưng theo reference thì chính sách v2024 hiện hành quy định 15 ngày cơ bản. Answer dùng chính sách v2023 đã bị thay thế nên không đúng phiên bản. | 0 | True |
| 46 | Nhân viên thử việc có được nghỉ phép năm không? | Nhân viên thử việc không được nghỉ phép năm và phải xin nghỉ không lương nếu cần. | 1 | Answer trả lời đúng theo reference: nhân viên thử việc không được nghỉ phép năm; nếu cần nghỉ thì phải xin nghỉ không lương. Trùng nội dung phủ định và điều kiện quan trọng. | 1 | True |
| 50 | Nhân viên Manager có thể dùng VPN cá nhân (NordVPN) khi WFH không? | Được, miễn là đảm bảo kết nối an toàn. | 0 | Answer nói rằng nhân viên có thể dùng VPN cá nhân miễn an toàn, nhưng reference nêu rõ chính sách VPN v1.3 cấm sử dụng VPN cá nhân và bắt buộc dùng VPN WireGuard của công ty. | 0 | True |

Cohen’s κ: 1.0.
Observed/chance agreement và diễn giải:
- Số câu quan sát: 10 câu.
- Phân bố nhãn Human: 6 nhãn 1 (Good), 4 nhãn 0 (Bad).
- Phân bố nhãn Judge: 6 nhãn 1 (Good), 4 nhãn 0 (Bad).
- Observed Agreement (Po) = 10 / 10 = 1.0 (100%).
- Chance Agreement (Pe) = (6/10 × 6/10) + (4/10 × 4/10) = 0.36 + 0.16 = 0.52.
- Hệ số Cohen's κ = (Po - Pe) / (1 - Pe) = (1.0 - 0.52) / (1.0 - 0.52) = 1.0.
Theo thang đo Landis & Koch (1977), hệ số κ = 1.0 đạt mức "Hoàn hảo" (Almost Perfect / Perfect Agreement), phản ánh mức độ trùng khớp tuyệt đối giữa quyết định đánh giá của LLM Judge và chuyên gia con người trên tập mẫu kiểm định.

## Giới hạn và cách dùng judge

- Giới hạn về tập dữ liệu và mô hình:
  + Kích thước tập mẫu đánh giá nhãn người gồm 10 câu và 5 cặp so sánh pairwise là quy mô kiểm chuẩn ban đầu, cần mở rộng khi đưa vào production toàn diện.
  + Mô hình Judge sử dụng `gpt-5.4-nano` với system prompt nghiêm ngặt dựa trên ground-truth reference. Khi áp dụng trong môi trường production không có reference sẵn (reference-free evaluation), độ chính xác của Judge có thể bị ảnh hưởng.
- Cách sử dụng LLM Judge trong quy trình CI/CD:
  + Không bao giờ coi kết quả của LLM Judge là chân lý tuyệt đối thay thế hoàn toàn con người. Cần duy trì cơ chế Human-in-the-Loop (HITL) để định kỳ lấy mẫu ngẫu nhiên (auditing sampling 5-10%) và kiểm chứng lại các quyết định của Judge.
  + Bắt buộc áp dụng kỹ thuật swap-and-average trong toàn bộ các quy trình pairwise testing nhằm triệt tiêu hoàn toàn Position Bias (như đã ghi nhận 40.0% trong đợt thử nghiệm này).
  + Khi phát hiện trường hợp lệch nhãn hoặc điểm số giữa hai lượt có độ chênh lệch lớn, hệ thống cần gắn cờ cảnh báo (flagging) để chuyển sang chuyên gia nhân sự/CNTT phúc tra thủ công trước khi phê duyệt release pipeline mới.
