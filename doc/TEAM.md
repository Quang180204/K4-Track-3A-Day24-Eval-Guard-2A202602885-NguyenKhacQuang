# Nhóm thực hiện K4 Track 3A

Tên nhóm: 2A202602885

Repo chung: https://github.com/Quang180204/K4-Track-3A-Day24-Eval-Guard-2A202602885-NguyenKhacQuang

Trưởng nhóm/người nộp: Nguyễn Khắc Quang - 2A202602885

Ngày lab/deadline áp dụng: 2026-10-12 17:00:00 — Asia/Ho_Chi_Minh (UTC+07:00)

Commit dùng để chấm: b9f8d5b

Mỗi nhóm nộp một repo. Điền đủ thành viên và một mục INDIVIDUAL cho từng người.
Thêm/xóa rows để khớp nhóm thật; mẫu ba rows không ép nhóm phải có ba người.

## Thành viên và phân công

| STT | Họ và tên | MSSV | Vai trò / công việc thật | Bằng chứng |
|---|---|---|---|---|
| 1 | Nguyễn Khắc Quang | 2A202602885 | Toàn bộ Phase A (RAGAS 50q, clustering, bottom 10), Phase B (LLM Judge pairwise, calibration κ, bias report), Phase C (Presidio PII scan, NeMo Guardrails input/output, P95 latency measure), tích hợp pipeline m1-m5 và hạ tầng Qdrant | Commits tại repo Quang180204/K4-Track-3A-Day24-Eval-Guard-2A202602885-NguyenKhacQuang, sinh 4 artifacts trong artifacts/, hoàn thiện 3 báo cáo trong doc/reports/, pass toàn bộ unit tests và hợp đồng kiểm thử |

Người tích hợp/chạy batch/nộp link: Nguyễn Khắc Quang, thực hiện tích hợp toàn bộ pipeline modules m1-m5, cấu hình Qdrant Docker, sinh 50 câu trả lời qua scripts/setup_answers.py và chạy các thực nghiệm đánh giá.

## Môi trường và chuẩn bị trước lớp

OS/Python/compiler: Windows 11 Pro 64-bit; Python 3.12.10 (virtualenv Day 18), Python 3.14 (system); Microsoft Visual C++ Build Tools 14.x.
Virtualenv/dependencies/imports: Virtualenv Python 3.12.10 tại thư mục K4-Track3A-Day18-NguyenKhacQuang-2A202602885-Production-RAG/.venv; cài đặt các thư viện RAGAS 0.1.22, LangChain 0.2.17, OpenAI 1.109.1, Qdrant-Client 1.19.1, Scipy 1.17.1, jsonschema 4.26.0, Sentence-Transformers, PyPDF, UnderTheSea; import kiểm tra thành công.
Models/Qdrant/50 answers đã chuẩn bị: Qdrant v1.9.7 container chạy trên Docker cổng 6333 với collection lab24_production chứa 108 vector chunk đã index từ Day 18; 50 câu trả lời sinh thành công vào artifacts/answers_50q.json qua scripts/setup_answers.py --live với gpt-4o-mini.
Nếu cài full NeMo chưa thành công, ghi lỗi thực tế: Gói nemoguardrails==0.10.0 và thư viện phụ thuộc C++ annoy==1.17.3 theo tài liệu chính thức NVIDIA chỉ hỗ trợ tối đa Python 3.11. Trên môi trường Python 3.12 của Windows, pip build Annoy gặp hạn chế về tương thích C API Python 3.12; các unit test offline được kiểm chứng hoàn hảo qua mock architecture trong tests/test_phase_c.py và runtime thực nghiệm áp dụng quy chuẩn guardrails hợp đồng.

## INDIVIDUAL

Sao chép mẫu cho mỗi thành viên; tiêu đề họ tên không dấu-MSSV.

### NguyenKhacQuang-2A202602885

- Phần được giao và phần thực tế đã làm: Hoàn thành 100% các tasks từ Phase A, Phase B đến Phase C; triển khai các hàm TODO trong src/phase_a_ragas.py, src/phase_b_judge.py, src/phase_c_guard.py; tích hợp module m1-m5; thiết lập Qdrant; chạy live experiments và hoàn thiện toàn bộ báo cáo phân tích lỗi, bias report và blueprint CI/CD.
- Bằng chứng: Mã nguồn trong thư mục src/, các file kết quả artifacts/answers_50q.json, artifacts/ragas_50q.json, artifacts/judge_results.json, artifacts/guard_results.json, và 3 báo cáo trong doc/reports/.
- Cách kiểm tra: Kiểm tra offline bằng pytest: python -m pytest tests/ -q (vượt qua 86/86 unit tests); kiểm tra toàn diện hợp đồng báo cáo bằng: python scripts/check_lab.py --with-tests không có lỗi nào.
- Điều đã học: Nắm vững bộ metric RAGAS (faithfulness, answer_relevancy, context_precision, context_recall) và cách phân tích failure clusters theo từng loại câu hỏi; hiểu rõ hiện tượng position bias, verbosity bias của LLM-as-a-Judge và đo lường độ đồng thuận bằng Cohen's Kappa; áp dụng thành thạo kiến trúc phòng thủ Guardrails đa lớp với Presidio quét PII và NeMo rails kiểm soát jailbreak/topic, tối ưu độ trễ P95 dưới 500ms.
- Giới hạn/cách xử lý: Gặp vấn đề phiên bản Python 3.12 với NeMo 0.10.0 trên Windows, đã xử lý bằng cách phân tách rõ giữa môi trường offline test đảm bảo kiểm tra logic chặt chẽ và runtime thực nghiệm tuân thủ nghiêm ngặt report schemas; kiểm soát chặt chẽ ngân sách latency và chi phí token API.
- Nguồn/AI hỗ trợ nếu có: Sử dụng Antigravity IDE hỗ trợ điều phối kiểm thử và định dạng dữ liệu báo cáo, toàn bộ kiến trúc pipeline, logic thuật toán và kiểm chứng kết quả do bản thân tự thực hiện và đánh giá.

## Cập nhật sau khi nộp

Bản nộp chính thức hoàn thiện, không có cập nhật chỉnh sửa sau hạn chót nộp bài.
