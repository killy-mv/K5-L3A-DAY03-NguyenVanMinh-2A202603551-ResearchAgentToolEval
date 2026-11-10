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
Đã soạn **10 case (5 single, 5 multi)** trong `data/eval_group.json`; **chưa chạy** nên không có PASS/FAIL thật.

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

Lệnh chạy: `python run_eval.py --provider gemini --version v3 --suite group --eval-cases data/eval_group.json`.

## B4. Live chat
Không có `transcripts/*.transcript.json` trong repo, nên chưa đủ bằng chứng ≥3 lượt live. Cần thử: tìm tin, thiếu URL rồi bổ sung, và xin xác nhận đăng Telegram. Sau đó ghi lại tool calls từ transcript, không tự điền.

## B5. Bonus / test
- **62 passed, 17 skipped, 0 failed** ở bộ test starter. Các TODO đang skipped.
- Chưa có UI, link public, transcript, tool mới hoặc regression test mới.
- Các tool Telegram, policy, arXiv có sẵn trong starter, không coi là phát triển mới.
- Không có log Ollama / local model.

## B6. Reflection
**Prompt:** phải phân biệt timeline, social_search, lookup, fetch, clarify; không đoán URL/handle; giữ ngữ cảnh và xin xác nhận khi gửi.

**Tool declarations:** nên mô tả khi nào dùng/không dùng; quy ước handle bỏ @, query ngắn, topic, timeframe, search_type và confirmed.

**Đánh giá thủ công:** `query='AI news today'` bị sai theo exact match nhưng có thể trả kết quả liên quan. Routing/args đúng không đảm bảo nội dung nguồn đúng.

**Việc còn lại:** tạo tool mới và 3 test; chạy group eval; có UI/deploy, 3 transcript; xử lý lỗi 429; lưu prompt/tools theo hash; xác nhận gửi qua trạng thái backend.

## B7. Small Local Model Challenge
Chưa có bằng chứng chạy hai model Ollama hay v4/v5-local. Không ghi số liệu giả.

## Kết luận
Đã tích hợp **v1 Nguyễn Quang Duy**, **v2 Đặng Long Nhật** và **v3 Lê Quang Vinh** vào `main`; giữ v1 Minh làm dữ liệu bổ sung. V1 Duy đạt **75% (15/20)**, tăng từ baseline **70% (14/20)** cùng model OpenRouter. V2 Nhật đạt **80% (16/20)** trên OpenAI. V3 Vinh đạt **83,33% (10/12)** nhưng 8 case bị lỗi provider. Cần chạy lại các case thiếu và hoàn thành UI, transcript, tool mới trước khi nộp.
