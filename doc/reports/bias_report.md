# Đánh giá LLM Judge Phase B

Nhóm: 2A202602885

Người phụ trách: Nguyễn Khắc Quang - 2A202602885

Nguồn: `artifacts/judge_results.json`; run timestamp: 2026-10-12T05:10:04.729826+00:00.
Models generation/judge: generation: ["openai/gpt-4o-mini"]; judge: ["gpt-4o-mini"].

## Năm cặp A/B và hai lượt

Nguồn cố định: A từ artifacts/answers_50q.json:answer; B từ fixtures/human_labels_10q.json:model_answer; reference từ fixtures/test_set_50q.json:ground_truth.
Ids: 1, 5, 12, 21, 41. Không đổi nguồn sau khi xem kết quả judge.

| ID | Candidate A | Candidate B | Winner Pass 1 | Winner Pass 2 | Final Winner | Position Consistent | Scores Pass 1 (A/B) | Scores Pass 2 (A/B) | Lý do chính |
|---|---|---|---|---|---|---|---|---|---|
| 1 | Nhân viên được nghỉ 3 ngày làm việc khi kết hôn. | Nhân viên được nghỉ 3 ngày làm việc có lương khi kết hôn. | B | B | B | Có (True) | 0.7 / 1.0 | 0.8 / 1.0 | Answer B nêu rõ nghỉ có lương, bảo đảm đầy đủ quyền lợi hơn A. |
| 5 | Cần phê duyệt của Trưởng phòng và Kế toán trưởng. | Cần Giám đốc phòng ban phê duyệt. | B | A | tie | Không (False) | 0.3 / 0.7 | 0.7 / 0.3 | Cả 2 câu trả lời đều sai (chuẩn là Tổng Giám đốc), thẩm phán ưu tiên ứng viên đứng trước ở mỗi lượt. |
| 12 | Thưởng Tết tối thiểu cho nhân viên chính thức có từ 6 tháng trở lên là **1 tháng lương**. | Nhân viên được thưởng Tết tối thiểu 1 tháng lương. | A | A | A | Có (True) | 1.0 / 0.8 | 1.0 / 0.8 | Answer A đầy đủ đối tượng điều kiện thâm niên 6 tháng hơn B. |
| 21 | Nhân viên Senior có 9 năm thâm niên được nghỉ 18 ngày phép năm (15 + 3). Lương của nhân viên này nằm trong khoảng có lương cho phép. | Được nghỉ 18 ngày phép (15 cơ bản + 3 thâm niên). Lương Senior từ 20 đến 35 triệu. | A | B | tie | Không (False) | 1.0 / 0.9 | 0.8 / 1.0 | Lượt 1 ưu tiên A, lượt 2 hoán đổi vị trí lại chấm B thắng do nêu rõ khoảng 20-35 triệu. |
| 41 | Mỗi nhân viên chính thức được hưởng **15 ngày phép năm** có lương. | Nhân viên được nghỉ 12 ngày phép năm. | A | A | A | Có (True) | 1.0 / 0.0 | 1.0 / 0.2 | Answer A đúng theo chính sách v2024 mới nhất, Answer B dùng chính sách v2023 cũ. |

Position bias count/rate: 2 câu / 0.4 (40.0%).
Verbosity win rate: 1.0 (100.0%); decisive cases: 3 câu.

Giải thích và phân tích nguyên nhân:
- Hiện tượng Position Bias: Xuất hiện ở 2/5 cặp câu hỏi (câu 5 và câu 21), chiếm tỷ lệ 40%. Ở câu 5, cả hai câu trả lời A và B đều đưa ra chức danh phê duyệt không đúng với quy định (chính sách yêu cầu Tổng Giám đốc). Khi cả hai phương án đều có khiếm khuyết, LLM Judge có xu hướng thiên vị lựa chọn câu trả lời xuất hiện ở vị trí đầu tiên trong prompt. Việc áp dụng phương pháp Swap-and-Average hoán đổi vị trí đã trung hòa bias này thành kết quả hòa (tie), ngăn chặn phán quyết sai lệch.
- Hiện tượng Verbosity: Trong 3 trường hợp phân định thắng bại dứt khoát (câu 1, 12, 41), câu trả lời dài hơn đều giành chiến thắng. Tuy nhiên, qua đối chiếu kỹ lưỡng nội dung và ground truth, câu dài hơn thực sự chứa các thông tin thiết yếu mà câu ngắn bỏ sót (câu 1 có cụm từ "có lương", câu 12 có điều kiện thâm niên "từ 6 tháng trở lên", câu 41 khẳng định số ngày 15 của v2024 thay vì 12 ngày của v2023). Do đó, sự thắng thế ở đây phản ánh tính đầy đủ về mặt ngữ nghĩa chứ không phải thiên vị hình thức đơn thuần.

## Calibration đúng model_answer

Helper đánh giá đúng question/model_answer đang có trong fixtures/human_labels_10q.json; không lấy answer mới của pipeline. Prompt chỉ nhận question, answer, reference. Không gửi human_label, human_note hoặc phân bố nhãn human vào prompt.

| ID | Model Answer Gốc | Nhãn Judge | Nhãn Human | Đồng Thuận | Reasoning của Thẩm Phán LLM |
|---|---|---|---|---|---|
| 1 | Nhân viên được nghỉ 3 ngày làm việc có lương khi kết hôn. | 1 | 1 | Đồng thuận | Câu trả lời chính xác theo thông tin trong tài liệu tham khảo, nêu rõ số ngày nghỉ và điều kiện không trừ vào phép năm. |
| 5 | Cần Giám đốc phòng ban phê duyệt. | 0 | 0 | Đồng thuận | Câu trả lời sai vì không nêu đúng người phê duyệt theo chính sách, cần là Tổng Giám đốc chứ không phải Giám đốc phòng ban. |
| 12 | Nhân viên được thưởng Tết tối thiểu 1 tháng lương. | 1 | 1 | Đồng thuận | Câu trả lời chính xác theo thông tin trong reference, nêu rõ điều kiện và mức thưởng cho nhân viên chính thức có từ 6 tháng trở lên. |
| 21 | Được nghỉ 18 ngày phép (15 cơ bản + 3 thâm niên). Lương Senior từ 20 đến 35 triệu. | 1 | 1 | Đồng thuận | Câu trả lời chính xác theo reference, nêu rõ số ngày phép và khoảng lương cho nhân viên Senior với 9 năm thâm niên. |
| 23 | Hoàn trả 25 triệu vì chưa đủ 1 năm cam kết. | 1 | 1 | Đồng thuận | Câu trả lời đúng với thông tin trong tài liệu tham khảo, nêu rõ rằng nhân viên phải hoàn trả 100% chi phí vì chưa đủ 1 năm cam kết. |
| 29 | Trưởng phòng phê duyệt. Phạt 2% tháng trên 8 triệu. | 0 | 0 | Đồng thuận | Câu trả lời sai vì không đề cập đến sự cần thiết có sự phê duyệt của Kế toán trưởng bên cạnh Trưởng phòng. Ngoài ra, phí phạt được tính cho quá hạn 15 ngày, không phải là 2% trên 8 triệu mà là 80.000 VNĐ. |
| 33 | Phép: 19 ngày. Phụ cấp: 1.500.000 VNĐ/tháng. | 1 | 1 | Đồng thuận | Câu trả lời cung cấp đúng số ngày phép và tổng phụ cấp hàng tháng theo thông tin trong tài liệu tham khảo. |
| 41 | Nhân viên được nghỉ 12 ngày phép năm. | 0 | 0 | Đồng thuận | Câu trả lời sai vì nó dựa trên chính sách cũ (v2023) thay vì chính sách hiện hành (v2024), trong đó quy định nhân viên được nghỉ 15 ngày phép năm. |
| 46 | Nhân viên thử việc không được nghỉ phép năm và phải xin nghỉ không lương nếu cần. | 1 | 1 | Đồng thuận | Câu trả lời chính xác theo reference, nêu rõ rằng nhân viên thử việc không được nghỉ phép năm và phải xin nghỉ không lương nếu cần, đồng thời không bỏ qua điều kiện cần có sự phê duyệt của trưởng phòng. |
| 50 | Được, miễn là đảm bảo kết nối an toàn. | 0 | 0 | Đồng thuận | Câu trả lời sai vì cho phép sử dụng VPN cá nhân trong khi chính sách rõ ràng cấm điều này. |

Cohen’s κ: 1.0.
Observed/chance agreement và diễn giải:
- Tỷ lệ đồng thuận quan sát ($P_o$): 10/10 câu = 1.0 (100%).
- Phân bố nhãn thực tế: 6 nhãn 1 (chấp nhận) và 4 nhãn 0 (từ chối).
- Xác suất đồng thuận ngẫu nhiên ($P_e$): $P_e = (0.6 \times 0.6) + (0.4 \times 0.4) = 0.36 + 0.16 = 0.52$.
- Tính toán chỉ số Cohen's Kappa: $\kappa = (P_o - P_e) / (1 - P_e) = (1.0 - 0.52) / (1 - 0.52) = 1.0$.
- Nhận xét: Chỉ số $\kappa = 1.0$ đạt mức "Almost Perfect Agreement" theo thang đo Landis & Koch. LLM Judge thể hiện khả năng phân biệt cực kỳ chính xác giữa thông tin đúng, thông tin thiếu người phê duyệt, lỗi tính toán tiền phạt và bẫy chính sách cũ.

## Giới hạn và cách dùng judge

Giới hạn và khuyến nghị áp dụng:
- Giới hạn kích thước mẫu: Bộ dữ liệu hiệu chuẩn 10 câu và 5 cặp so sánh là mẫu kiểm tra khái niệm (proof-of-concept). Mặc dù đạt $\kappa = 1.0$, trong môi trường vận hành quy mô lớn với hàng nghìn truy vấn đa dạng, sự tương quan có thể dao động trong khoảng 0.70 - 0.85.
- Thiết kế Prompt: Prompt judge binary chỉ tập trung vào tính chính xác của thông tin theo tài liệu tham khảo và không phạt phong cách hành văn. Điều này giúp thẩm phán nhất quán nhưng chưa đánh giá được độ thân thiện và văn phong chuyên nghiệp.
- Không xem LLM Judge là thẩm quyền tối cao: Luôn cần có cơ chế giám sát con người (Human-in-the-Loop) định kỳ. Đối với các câu hỏi nhạy cảm về pháp lý, kỷ luật hoặc sa thải, kết quả judge chỉ đóng vai trò phân loại sơ bộ, quyết định cuối cùng phải do chuyên viên nhân sự kiểm duyệt.
