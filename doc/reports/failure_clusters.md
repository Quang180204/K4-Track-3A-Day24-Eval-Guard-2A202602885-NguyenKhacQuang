# Phân tích lỗi Phase A

Nhóm: 2A202602885

Người phụ trách: Nguyễn Khắc Quang - 2A202602885

Nguồn: `artifacts/ragas_50q.json`; run timestamp: 2026-10-12T05:09:06.413990+00:00.
Generation mode/fallback và điều kiện chạy: Thực nghiệm live toàn bộ qua OpenAI API với model gpt-4o-mini; evaluator RAGAS 4 metrics kết hợp mô hình ngôn ngữ gpt-4o-mini và embedding all-MiniLM-L6-v2; hạ tầng vector search Qdrant v1.9.7 cổng 6333 collection lab24_production chứa 108 chunks đã index.
Models generation/evaluator: generation: ["openai/gpt-4o-mini"]; evaluator: ["all-MiniLM-L6-v2", "gpt-4o-mini"].

## Điểm theo distribution

| Metric | factual (20) | multi_hop (20) | adversarial (10) |
|---|---|---|---|
| faithfulness | 0.9750 | 0.4730 | 0.7000 |
| answer_relevancy | 0.1792 | 0.1208 | 0.2401 |
| context_precision | 0.8333 | 0.7208 | 0.6833 |
| context_recall | 0.7583 | 0.7292 | 0.7167 |
| avg_score | 0.6865 | 0.5110 | 0.5850 |

## Bottom 10 và nguyên nhân

| Rank | ID | Question | Distribution | Avg Score | Worst Metric | Diagnosis | Suggested Fix |
|---|---|---|---|---|---|---|---|
| 1 | 21 | Một nhân viên Senior có 9 năm thâm niên được nghỉ bao nhiêu ngày phép năm và lương trong khoảng nào? | multi_hop | 0.0333 | faithfulness | LLM hallucinating | Tighten system prompt, lower temperature |
| 2 | 38 | Nhân viên đi công tác nước ngoài 4 ngày tại thành phố lớn, ở khách sạn 200 USD/đêm (3 đêm). Công ty thanh toán bao nhiêu tiền khách sạn? | multi_hop | 0.0833 | context_precision | Too many irrelevant chunks | Add reranking or metadata filter |
| 3 | 30 | So sánh quyền lợi bảo hiểm giữa nhân viên thử việc và nhân viên chính thức. | multi_hop | 0.1250 | faithfulness | LLM hallucinating | Tighten system prompt, lower temperature |
| 4 | 46 | Nhân viên thử việc có được nghỉ phép năm không? | adversarial | 0.1667 | faithfulness | LLM hallucinating | Tighten system prompt, lower temperature |
| 5 | 12 | Thưởng Tết tối thiểu cho nhân viên chính thức có từ 6 tháng trở lên là bao nhiêu? | factual | 0.2500 | answer_relevancy | Answer doesn't match question | Improve prompt template |
| 6 | 20 | Phụ cấp đi lại tối đa mỗi tháng là bao nhiêu và điều kiện để được hưởng là gì? | factual | 0.2500 | answer_relevancy | Answer doesn't match question | Improve prompt template |
| 7 | 29 | Nhân viên tạm ứng 8 triệu, chưa thanh toán sau 30 ngày (quá hạn 15 ngày). Ai phê duyệt khoản này và phí phạt là bao nhiêu? | multi_hop | 0.3125 | answer_relevancy | Answer doesn't match question | Improve prompt template |
| 8 | 47 | Khi phát hiện malware trên máy tính công ty, nhân viên có nên tự xử lý không? | adversarial | 0.3158 | faithfulness | LLM hallucinating | Tighten system prompt, lower temperature |
| 9 | 22 | Nếu cần mua một chiếc laptop 30 triệu cho nhân viên mới, ai phê duyệt và cần gì từ phòng CNTT? | multi_hop | 0.3333 | faithfulness | LLM hallucinating | Tighten system prompt, lower temperature |
| 10 | 33 | Nhân viên Manager có thâm niên 12 năm: tổng phụ cấp hàng tháng và số ngày phép năm theo v2024 là bao nhiêu? | multi_hop | 0.3333 | faithfulness | LLM hallucinating | Tighten system prompt, lower temperature |

Đối chiếu chi tiết các nhóm câu hỏi đại diện:
- Câu 21 (multi_hop): Yêu cầu tích hợp cả 2 nguồn văn bản là chính sách nghỉ phép (15 ngày cơ bản + 3 ngày cho 9 năm thâm niên = 18 ngày) và chính sách lương (khung lương Senior: 20 - 35 triệu). Tuy nhiên retrieval chỉ trả về các đoạn liên quan đến phép năm, thiếu thông tin bảng lương. Do thiếu context cần thiết, LLM sinh câu trả lời mập mờ "nằm trong khoảng có lương cho phép", dẫn tới faithfulness đạt 0.0. Đây là lỗi điển hình của retrieval đa chặng khi thiếu reranking đa nguồn.
- Câu 38 (multi_hop): Câu hỏi tính toán mức trần công tác phí khách sạn thành phố lớn (150 USD/đêm x 3 đêm = 450 USD thay vì 200 USD/đêm). Retrieval trả về nhiều chunk phụ cấp ngày trong nước và thủ tục thanh toán không liên quan, làm context_precision tụt xuống 0.33. LLM dù tính đúng nhưng context chứa quá nhiều nhiễu.
- Câu 46 (adversarial): Câu hỏi bẫy phủ định về việc nhân viên thử việc có được nghỉ phép năm hay không. Tài liệu quy định rõ "KHÔNG được nghỉ phép năm". Dù answer nêu không được nghỉ phép năm nhưng việc đối chiếu factual bị lệch do RAGAS trích xuất statement chưa tương thích hoàn toàn giữa tiếng Việt và embedding tiếng Anh, làm điểm faithfulness bị chấm thấp.

## Matrix và nhóm yếu nhất

| Metric | factual (20) | multi_hop (20) | adversarial (10) |
|---|---|---|---|
| faithfulness | 0 | 6 | 3 |
| context_recall | 2 | 0 | 1 |
| context_precision | 0 | 2 | 1 |
| answer_relevancy | 18 | 12 | 5 |

Nhóm yếu theo mean thấp nhất: multi_hop; mean: 0.5110.
Worst metric xuất hiện nhiều nhất: answer_relevancy; số câu: 35.

Nguyên nhân và giải thích:
- Nhóm multi_hop có mean thấp nhất (0.5110) và faithfulness thấp kỷ lục (0.4730 so với 0.9750 của factual). Lý do cốt lõi là các câu hỏi đa bước đòi hỏi kết hợp đồng thời từ 2 văn bản chính sách khác nhau (ví dụ: thâm niên + thang bảng lương, công tác phí + quy định khách sạn). Hybrid search độc lập từng chunk không đảm bảo retrieval bao phủ đầy đủ tất cả các thực thể cần thiết nếu không có kỹ thuật query decomposition hoặc multi-query retrieval.
- Metric answer_relevancy xuất hiện nhiều nhất trong vị trí worst_metric (35 câu trên tổng 50 câu). Nguyên nhân là câu trả lời của mô hình thường trực tiếp và ngắn gọn (1-2 câu), trong khi thuật toán answer_relevancy của RAGAS sinh ngược lại các câu hỏi giả định từ câu trả lời bằng all-MiniLM-L6-v2 rồi tính cosine similarity. Khi câu trả lời quá ngắn gọn bằng tiếng Việt, các câu hỏi tái tạo từ embedding tiếng Anh bị suy giảm độ tương đồng ngữ nghĩa.
- Đề xuất sửa đổi cụ thể: Bổ sung bước Query Expansion / Multi-Query Generation trước khi tìm kiếm; tăng top_k retrieval từ 3 lên 6 và dùng FlashRank reranker để lọc các chunk sát nghĩa nhất; tinh chỉnh prompt template yêu cầu trả lời mạch lạc theo cấu trúc Chain-of-Thought từng bước. Chạy lại kiểm chứng bằng lệnh `python scripts/run_experiments.py --phase a --live` để đối chiếu cải thiện của avg_score.

## Bẫy adversarial và giới hạn

So sánh với factual và multi_hop:
- Các câu hỏi adversarial (như câu 41 về số ngày phép theo chính sách cũ v2023 là 12 ngày so với v2024 là 15 ngày, câu 46 về phép năm thử việc, câu 47 về tự ý diệt malware) đưa ra các bẫy về phiên bản văn bản hết hiệu lực và tiền đề phủ định.
- Điểm context_precision của adversarial thấp nhất (0.6833) và faithfulness chỉ đạt 0.7000 (thấp hơn nhiều so với factual 0.9750). Điều này cho thấy khi trong kho dữ liệu tồn tại cả tài liệu phiên bản cũ và phiên bản mới, vector search thuần túy rất dễ lấy nhầm chunk cũ có độ tương đồng từ khóa cao nếu không áp dụng metadata filtering theo trường ngày hiệu lực hoặc trạng thái tài liệu.
- Giới hạn thực nghiệm: Tập dữ liệu gồm 50 câu hỏi phân bố 20 factual, 20 multi_hop, 10 adversarial; mô hình LLM evaluator là gpt-4o-mini qua API nước ngoài có thể chịu ảnh hưởng bởi độ trễ mạng và giới hạn rate limit; embedding all-MiniLM-L6-v2 là mô hình tiếng Anh nên độ chính xác trong đo lường answer_relevancy cho văn bản tiếng Việt còn có khoảng lệch nhất định.
