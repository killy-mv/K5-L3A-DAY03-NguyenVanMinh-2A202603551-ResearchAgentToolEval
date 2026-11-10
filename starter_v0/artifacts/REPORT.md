# Báo cáo Lab 3 — Research Agent Tool Evaluation

## Thông tin nhóm
**Nhóm:** Halluciation. **Thành viên:** Nguyễn Văn Minh (03551), Lê Quang Vinh (03363), Nguyễn Quang Duy (034462), Đặng Long Nhật (03483), Nguyễn Đức Đạt (03544). **Các mốc:** v1 Nguyễn Quang Duy (chính), v2 Đặng Long Nhật, v3 Lê Quang Vinh. Kết quả v1 Nguyễn Văn Minh được giữ làm bằng chứng bổ sung.

# PHẦN A — GIỚI THIỆU AGENT

## A1. Chức năng
Research Agent chọn tool để tìm tin trên web, lấy bài đăng theo tài khoản/chủ đề, đọc URL và tóm tắt. Nếu thiếu dữ liệu bắt buộc, cần dùng `clarify` thay vì đoán. Trước khi đăng Telegram phải xin xác nhận. **Link deploy UI:** chưa có trong repo. Có thể chạy CLI bằng `python chat.py --provider gemini --version v3` trong `starter_v0`.

## A2. Công cụ
| Tool | Chức năng | Tự thêm? |
|---|---|---|
| clarify | Hỏi bổ sung hoặc xác nhận | Không |
| timeline | Lấy bài đăng tài khoản cụ thể | Không |
| social_search | Tìm bài đăng theo chủ đề, Top/Latest | Không |
| lookup | Tìm thông tin web/tin tức | Không |
| fetch | Đọc URL | Không |
| format | Trình bày digest | Không |
| send | Gửi Telegram sau xác nhận | Không |
| policy | Tra cứu policy nội bộ | Không |
| papers | Tìm paper arXiv | Không |
| paper_text | Đọc text paper arXiv | Không |

**Chưa hoàn thành:** tool mới do nhóm viết và ít nhất 3 unit test.

## A3. Câu hỏi mẫu
1. Lấy 5 tweet gần nhất của Elon Musk.
2. Tin AI hôm nay có gì nổi bật?
3. Mọi người bàn gì về robotics trên Twitter?
4. Tóm tắt https://example.com/research.
5. Đăng bản tin này lên Telegram (cần xin xác nhận).

# PHẦN B — KẾT QUẢ VÀ BẰNG CHỨNG

## B1. Đánh giá phiên bản

| Bản / tác giả | Thay đổi, giả thuyết | Kết quả chạy thật | File JSON trong `runs/` |
|---|---|---|---|
| v0 Minh / Gemini | Prompt gốc khuyến khích đoán | **9/18 = 50%**, 2 lỗi API; routing 66,67%; args 50%; multiturn 66,67% | `v0_B_base_gemini_20261110T105526641412.json` |
| **v1 Nguyễn Quang Duy / OpenRouter gpt-4o-mini** | Sửa prompt để hỏi lại khi thiếu handle/URL | **15/20 = 75%**, 0 lỗi provider; routing 90%, arguments 75%, multiturn 83,33% | `v1_B_base_openrouter_20261110T102355259578.json` |
| v1 Minh / Gemini (additional evidence) | Thêm `clarify` khi thiếu account/URL | **3/3 = 100%** trên mẫu hợp lệ, **17/20 lỗi API**; R10/R11 chưa kiểm chứng | `v1_B_base_gemini_20261110T113147651924.json` |
| v0 Nhật / OpenAI | Đối chứng v2 | **14/20 = 70%**, không có lỗi API; routing 75%; args 70%; multiturn 100% | `v0_B_base_openai_20261110T102042922136.json` |
| **v2 Nhật / OpenAI** | Tối ưu định tuyến | **16/20 = 80%**, không lỗi API; routing 85%; args 80%; multiturn 83,33% | `v2_B_base_openai_20261110T105051589517.json` |
| v2 Vinh / OpenRouter free | Mốc trước v3 | **14/20 = 70%**, không lỗi API | `v2_B_base_openrouter_20261110T111514782695.json` |
| **v3 Vinh / OpenRouter free** | Prompt routing chi tiết và `tools.yaml` rõ ràng | **10/12 = 83,33%**, **8/20 lỗi API**; routing 100%; args 83,33% | `v3_B_base_openrouter_20261110T112109678983.json` |

**Diễn giải:** Trên cùng OpenRouter gpt-4o-mini, v0 của Duy đạt 14/20 (70%) và v1 đạt 15/20 (75%), không có lỗi provider. Kết quả v1 Minh và v3 Vinh có nhiều lỗi API nên không so sánh trực tiếp. Phiên bản v2 của Nhật dùng OpenAI, cũng là provider khác.

**Giới hạn tái lập:** Run v2 Nhật ghi `prompt_hash=e67875...` nhưng snapshot prompt tương ứng không có trong Git. Run v3 Vinh ghi hash `47d17...` (prompt), `f0352...` (tools), khác các file v3 có trong Git (`9f007...`, `8b728...`). Đây là log lịch sử; muốn tái lập phải lưu và chạy lại đúng artifact.

## B2. Failure Analysis
| Case / bản | Sai lệch | Lý do và hướng khắc phục |
|---|---|---|
| R10 / Minh v0 | `timeline(sama)` thay `clarify` | Thiếu account nhưng tự đoán; v1 thêm rule hỏi lại, R10 v1 bị 429 |
| R11 / Minh v0 | `fetch` URL tự bịa | Thiếu URL; v1 yêu cầu hỏi URL, R11 v1 bị 429 |
| R12 / Minh v0 | `send(confirmed=true)` | Tự xác nhận; phải hỏi `clarify(yes_no)` trước |
| R10/R11 / Duy v1 | Đã chọn `clarify` nhưng thiếu `response_type=text` | Cần quy định đầy đủ arguments trong prompt và tool declarations |
| R01 / Nhật v2 | `clarify` thay `timeline` | Hỏi thừa về tài khoản Sam Altman, nên map `sama` |
| R13 / Nhật v2 | `lookup` thiếu `topic=news` | Cần chuẩn hóa tham số tin tức |
| M06 / Nhật v2 | `social_search` thay `lookup` | Không chuyển công cụ theo lượt cuối |
| R03 / Vinh v3 | query `AI news today` thay `AI` | Không tách thời gian khỏi chủ đề |
| R12 / Vinh v3 | `clarify(text)` thay `clarify(yes_no)` | Chọn đúng tool nhưng sai kiểu xác nhận |

**Rủi ro:** `send` kiểm tra cờ `confirmed`, nhưng LLM từng tự gán true. Xác nhận thực tế cần được lưu ở backend, không chỉ dựa vào tham số model.

## B3. Group Eval
Đã soạn **10 case (5 single, 5 multi)** trong `data/eval_group.json`; **đã chạy** bằng Gemini, 10/10 PASS.

| ID | Loại | Kỳ vọng |
|---|---|---|
| G01 | Single: Elon Musk 4 bài | timeline(elonmusk,4) |
| G02 | Single: bài phổ biến robotics | social_search Top |
| G03 | Single: thiếu URL | clarify text |
| G04 | Single: đăng Telegram | clarify yes_no |
| G05 | Single: tin AI hôm nay | lookup AI/news/day |
| G06 | Multi: đổi 10 thành 3 bài | timeline(elonmusk,3) |
| G07 | Multi: đổi robotics thành AI | lookup AI/news/week |
| G08 | Multi: thêm URL lượt sau | fetch URL |
| G09 | Multi: sửa Sam Altman thành Karpathy | timeline(karpathy,5) |
| G10 | Multi: chuyển Twitter thành web | lookup AI/news/day |

**Kết quả:** 10/10 PASS (100% case accuracy); tool routing 100%; argument accuracy 100%; multi-turn accuracy 100%; 0 provider errors.

**Bằng chứng:** `runs/v3_B_group_gemini_20261110T135959480307.json` (v3, Gemini flash-lite). Group eval đo tool calls và arguments do Gemini thật sinh ra. Việc thực thi HTTP/Telegram tool được mô phỏng/không gọi thật để tránh tác động ngoài và timeout. Do đó kết quả **không chứng minh** end-to-end functionality hay nội dung API trả về.

**Lệnh tham chiếu:** `python run_eval.py --provider gemini --version v3 --suite group --eval-cases data/eval_group.json` (lệnh gốc sẽ thực thi tool thật, khác với run routing-only này).

## B4. Live Chat Evidence

**Đã hoàn thành live chat thực tế:** một phiên gồm **4 lượt liên tiếp** trên Agent v3, sử dụng **Gemini (gemini-3.5-flash-lite)**. Dữ liệu lấy trực tiếp từ file `transcripts/v3_gemini_20261110T141111080092.transcript.json`, không phải transcript mô phỏng hoặc kết quả group eval.

- **Thời gian:** 2026-11-10T14:11:11 đến 2026-11-10T14:13:41 (giờ máy chạy).
- **Artifact version:** `v3+p7b37bf434ab1+t91db2613ae08`.
- **Trạng thái:** 2 lượt `answered` và 2 lượt `waiting_for_user`; không ghi nhận `provider_error`. Cả bốn lượt thể hiện đúng hành vi routing/confirmation kỳ vọng.

| Lượt | Yêu cầu thực tế | Tool calls / arguments thực tế | Status | Kết quả quan sát |
|---|---|---|---|---|
| 1 | Tìm 3 tin tức mới nhất về AI trong hôm nay trên web | `lookup(query="AI", topic="news", timeframe="day", max_results=3)` → `format` | `answered` | `lookup` trả 3 item; `format` tạo bản tin 3 mục, Agent phản hồi bằng văn bản |
| 2 | Tóm tắt bài viết này giúp tôi (chưa cung cấp URL) | `clarify(response_type="text")` | `waiting_for_user` | Agent hỏi URL cụ thể, không tự tạo URL hoặc gọi `fetch` |
| 3 | Đây là URL: https://www.python.org/about/ | `fetch(url="https://www.python.org/about/")` → `format` | `answered` | Tool `fetch` trả 1 trang; Agent tạo bản tóm tắt về Python.org |
| 4 | Đăng bản tóm tắt này lên Telegram giúp tôi | `clarify(response_type="yes_no")` | `waiting_for_user` | Agent hỏi xác nhận trước khi đăng. **Không có lời xác nhận hoặc lệnh `send`** trong phiên này |

**Nhận xét:** Các tool `lookup`, `fetch` và `format` được thực thi trong phiên chat thật, khác với group eval B3 chỉ kiểm tra routing và mô phỏng thực thi tool. Lượt 2–3 cho thấy Agent giữ được yêu cầu tóm tắt qua các lượt liên tiếp. Lượt 4 cho thấy ranh giới xác nhận được thực thi đúng ở thời điểm kiểm tra; **chưa kiểm chứng việc gửi Telegram sau khi người dùng xác nhận**.

**Giới hạn:** Transcript chứng minh các tool đã được gọi và phản hồi của chúng được dùng để tạo văn bản; chưa có kiểm chứng độc lập tính chính xác hoặc độ tin cậy của mọi nguồn tin. Không có bằng chứng về một lần gửi Telegram thành công.

## B5. Bonus / test
- **62 passed, 17 skipped, 0 failed** ở bộ test starter. Các TODO đang skipped.
- Chưa có UI/link public, tool mới hoặc regression test mới; **đã có transcript live 4 lượt** tại `transcripts/v3_gemini_20261110T141111080092.transcript.json`.
- Các tool Telegram, policy, arXiv có sẵn trong starter, không coi là phát triển mới.
- Không có log Ollama / local model.

## B6. Reflection
**Prompt:** phải phân biệt timeline, social_search, lookup, fetch, clarify; không đoán URL/handle; giữ ngữ cảnh và xin xác nhận khi gửi.

**Tool declarations:** nên mô tả khi nào dùng/không dùng; quy ước handle bỏ @, query ngắn, topic, timeframe, search_type và confirmed.

**Đánh giá thủ công:** `query='AI news today'` bị sai theo exact match nhưng có thể trả kết quả liên quan. Routing/args đúng không đảm bảo nội dung nguồn đúng.

**Việc còn lại:** tạo tool mới và 3 test; kiểm chứng end-to-end theo mục tiêu lab; có UI/deploy; xử lý lỗi 429; lưu prompt/tools theo hash; xác nhận gửi qua trạng thái backend. **Live chat 4 lượt đã hoàn thành và ghi nhận tại B4.**

## B7. Small Local Model Challenge
Chưa có bằng chứng chạy hai model Ollama hay v4/v5-local. Không ghi số liệu giả.

## Kết luận
Đã tích hợp **v1 Nguyễn Quang Duy**, **v2 Đặng Long Nhật** và **v3 Lê Quang Vinh** vào `main`; giữ v1 Minh làm dữ liệu bổ sung. V1 Duy đạt **75% (15/20)**, tăng từ baseline **70% (14/20)** cùng model OpenRouter. V2 Nhật đạt **80% (16/20)** trên OpenAI. V3 Vinh đạt **83,33% (10/12)** nhưng 8 case bị lỗi provider. Cần chạy lại các case thiếu và hoàn thành UI và tool mới trước khi nộp.
