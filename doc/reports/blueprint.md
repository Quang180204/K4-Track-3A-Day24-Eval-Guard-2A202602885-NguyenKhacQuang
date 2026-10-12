# Blueprint CI/CD cho RAG và guardrails

Nhóm: 2A202602885

Người phụ trách: Nguyễn Khắc Quang - 2A202602885

Đây là **đề xuất CI/CD**, không phải workflow đã được triển khai/chạy.
Nguồn kết quả: artifacts/ragas_50q.json, judge_results.json, guard_results.json.
Run timestamp: 2026-10-12T05:24:07.728956+00:00; guard models: ["gpt-4o-mini"].
Models generation/evaluator/judge từ A/B: generation: ["openai/gpt-4o-mini"] (nguồn: ragas_50q.json, judge_results.json); evaluator: ["all-MiniLM-L6-v2", "gpt-4o-mini"] (nguồn: ragas_50q.json); judge: ["gpt-4o-mini"] (nguồn: judge_results.json); guard: ["gpt-4o-mini"] (nguồn: guard_results.json).

## Kiến trúc và failure action

Input → Presidio → NeMo input → RAG → NeMo output → response.
Request thực dừng sớm khi bị chặn; khác phép đo input layers chạy đủ hai tầng.

Mô tả luồng dữ liệu và cơ chế bảo vệ:
- Luồng dữ liệu (Data flow): Yêu cầu từ người dùng trước tiên đi qua tầng quét thực thể nhạy cảm Presidio PII. Nếu an toàn, văn bản tiếp tục qua NeMo input rails để phát hiện jailbreak, prompt injection hoặc câu hỏi lạc đề. Chỉ các truy vấn hợp lệ mới được kích hoạt qua pipeline RAG (tìm kiếm Qdrant hybrid search, rerank FlashRank và LLM generation). Phản hồi từ mô hình trước khi gửi lại người dùng phải qua NeMo output rails nhằm ngăn ngừa rò rỉ dữ liệu mật hoặc PII phát sinh.
- Cơ chế dừng sớm (Short-circuit): Khác với bài đo latency Task 12 chạy bắt buộc tuần tự đủ 2 lớp để thu thập số đo, trong luồng phục vụ thực tế (production runtime), nếu bất kỳ lớp nào phát hiện vi phạm, hệ thống dừng xử lý ngay lập tức (early exit), trả về câu thông báo từ chối an toàn để tiết kiệm chi phí token và tài nguyên tính toán.
- Nhật ký và ẩn danh hóa (Log/Redact): Tất cả log truy vấn đều phải đi qua bộ lọc Presidio để che giấu PII (CCCD, SĐT, email cá nhân) trước khi lưu vào hệ thống giám sát tập trung. Nghiêm cấm ghi plain-text thông tin nhạy cảm vào log.
- Quản lý Secrets và Fallback: Khóa API, mật khẩu cơ sở dữ liệu và token Qdrant được lưu trữ an toàn trong Secret Manager (Vault/K8s Secrets) và inject qua biến môi trường tại runtime; tuyệt đối không lưu trong mã nguồn. Nếu downstream API gặp lỗi timeout (> 3s) hoặc cạn quota, hệ thống tự động kích hoạt fallback sang câu trả lời định sẵn hướng dẫn liên hệ bộ phận hỗ trợ helpdesk@cty.vn.

## Phạm vi và số đo latency

Bắt buộc Task 12: **input_layers_sequential**, Presidio + NeMo input, không
short-circuit. Engines tạo trước đo; có hai warm-up thật không tính vào mẫu.
Percentile nội suy tuyến tính từ raw samples; tổng từ tổng từng sample.

Số mẫu timed: 20; warm-up: 2;
phương pháp: linear.

| Layer / scope | P50 ms | P95 ms | P99 ms | Tình trạng |
|---|---|---|---|---|
| Presidio | 0.68 | 2.60 | 11.24 | Đạt hiệu năng cực tốt dưới 3ms |
| NeMo input | 1712.49 | 2304.01 | 2341.14 | Vượt budget do phụ thuộc round-trip API ngoài |
| Input guard sequential | 1712.91 | 2305.03 | 2342.37 | Đo tuần tự 2 tầng theo đúng Task 12 |
| RAG | chưa đo | chưa đo | chưa đo | Ngoài phạm vi bắt buộc |
| NeMo output | chưa đo | chưa đo | chưa đo | Ngoài phạm vi bắt buộc |
| Full guard / short-circuit request | chưa đo | chưa đo | chưa đo | Không suy từ input_guard_ms |

Input-layer budget target: 500ms; kết quả thực tế/budget_ok: 2305.03 ms và False.
Phân tích điểm nghẽn và giải pháp:
- Điểm sáng: Tầng Presidio PII hoạt động cực kỳ ấn tượng với P50 là 0.68 ms và P95 là 2.60 ms, hoàn toàn đáp ứng tiêu chuẩn sản xuất nhờ sử dụng engine regex tối ưu trên CPU.
- Điểm nghẽn (bottleneck): Tầng NeMo input rails có P95 lên tới 2304.01 ms, dẫn tới tổng độ trễ tuần tự đạt 2305.03 ms, vượt ngân sách 500ms. Nguyên nhân trực tiếp là trong điều kiện bài lab, việc kiểm tra self-check input rail phụ thuộc vào lệnh gọi mạng API qua Internet tới máy chủ OpenAI/OpenRouter thay vì chạy local inference. Độ trễ đường truyền mạng quốc tế (~1.5s - 2.3s) là yếu tố quyết định khiến thời gian đo đạc bị kéo dài.
- Giải pháp production: Triển khai một mô hình guardrail chuyên biệt kích thước nhỏ (như Llama-Guard-3-1B hoặc ONNX classifier) trực tiếp trong cùng mạng nội bộ (on-premise / VPC) với cụm inference vLLM. Khi đó, thời gian suy luận self-check input rail giảm xuống dưới 30ms, đưa toàn bộ tầng guardrail đầu vào về dưới 35ms, đáp ứng ngân sách 500ms một cách vượt trội.
- Đối với các phần ngoài phạm vi bắt buộc (RAG, NeMo output, full short-circuit request), giữ nguyên trạng thái "chưa đo"; trong lộ trình nâng cấp hạ tầng sẽ tích hợp OpenTelemetry Distributed Tracing để ghi nhận thời gian từng span chính xác.

## CI gates đề xuất

- RAGAS faithfulness ≥0.75, avg_score ≥0.65 trên 50 câu.
- Attacks ≥18/20 cho CI đề xuất; clean 8/8 allowed; output cases đúng.
- Full guard P95 <500ms chỉ áp dụng sau khi thực sự đo full guard.

Mốc bắt buộc lab vẫn attacks ≥15/20; bonus ≥18/20 kèm clean controls đạt.
Quy trình kiểm tra chất lượng và tích hợp:
- Đề xuất lệnh thực thi trong CI Pipeline:
```bash
python scripts/run_experiments.py --phase all --live
python scripts/check_lab.py --with-tests
```
- Cơ chế fail-fast: Script kiểm thử trả về mã lỗi khác 0 (nonzero returncode) khi bất kỳ tiêu chí chất lượng nào không đạt ngưỡng, lập tức chặn tiến trình build và không cho phép merge Pull Request vào nhánh chính (main).
- Lưu ý: Đây là đề xuất quy trình kiểm chuẩn CI/CD tự động phục vụ kế hoạch triển khai dài hạn, chưa được kích hoạt thành luồng chạy thực tế trên GitHub Actions tại bài lab này.

## Monitoring và cải tiến

Chiến lược giám sát vận hành:
- Giám sát theo thời gian thực (Real-time monitoring): Thu thập các chỉ số qua Prometheus và hiển thị trên Grafana dashboard, bao gồm: tỷ lệ chặn request của từng tầng guardrail (Presidio vs NeMo), tỷ lệ phản hồi tiêu cực của người dùng (user dislike/feedback), phân bố độ trễ P50/P90/P99 theo thời gian thực.
- Cảnh báo tự động: Kích hoạt cảnh báo mức độ cao (PagerDuty/Slack) khi tỷ lệ chặn bất thường vượt quá 20% trong khung thời gian 15 phút (nghi ngờ tấn công DoS hoặc false-positive diện rộng do thay đổi prompt), hoặc khi tỷ lệ lỗi kết nối API bên thứ ba vượt quá 5%.
- Bảo mật quyền riêng tư: Định kỳ kiểm toán ngẫu nhiên 1% dữ liệu truy vấn đã được redact để xác nhận không còn hiện tượng rò rỉ PII lọt qua hệ thống.

Các thông số thực tế đo lường được:
RAGAS mean/worst distribution: Factual đạt 0.6865, Multi-hop thấp nhất đạt 0.5110 (worst distribution).
Cohen’s κ: 1.0 (sự đồng thuận hoàn hảo giữa thẩm phán AI và nhãn con người).
Attacks passed: 18/20; clean false_positive_rate: 0.0 (đạt chuẩn tuyệt đối 8/8 câu sạch được cho phép).

Bốn đề xuất cải tiến trọng tâm có thể kiểm chứng:
1. Tối ưu hóa độ trễ Guardrail: Thay thế API gọi ra Internet bằng mô hình cục bộ ONNX / TensorRT-LLM để đưa P95 latency từ 2305 ms xuống dưới 50 ms.
2. Cải thiện chất lượng truy xuất đa chặng (Multi-hop Retrieval): Nâng số lượng chunk lấy về từ top_k=3 lên top_k=5 kết hợp FlashRank Reranker để nâng điểm faithfulness của nhóm multi-hop từ 0.4730 lên tối thiểu 0.7500.
3. Bộ lọc thời gian và phiên bản tài liệu (Metadata Filtering): Thêm điều kiện lọc `effective_date` và `is_active` vào truy vấn Qdrant để triệt tiêu việc lấy nhầm văn bản quy định cũ (v2023) trong các câu hỏi adversarial.
4. Mở rộng tập dữ liệu kiểm chuẩn: Tăng quy mô tập test từ 50 câu lên 300 câu bao quát toàn bộ 15 phòng ban chức năng, thiết lập quy trình đối soát định kỳ hàng tháng giữa đội ngũ AI và chuyên viên nhân sự.
