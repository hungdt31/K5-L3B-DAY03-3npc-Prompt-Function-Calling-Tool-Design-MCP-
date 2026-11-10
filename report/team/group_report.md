# Báo cáo nhóm — Day 03 Research Agent Tool Eval

> Báo cáo gồm 2 phần có hạn khác nhau:
> - **Phần A (mục 2)** — giới thiệu agent, ngắn gọn 1 trang. **Xong trước Team showdown (16:30)** để team khác hiểu nhanh agent có tool gì, làm được gì. Có thể làm thêm poster `report/team/poster.html` hoặc `poster.svg`.
> - **Phần B (mục 3–7)** — chi tiết và bằng chứng, dựa trên log thật. Hoàn thiện trước deadline nộp bài.
>
> Mỗi bảng phải trỏ được tới file thật: run JSON, transcript, commit. Cách chấm xem [RUBRIC.md](../../RUBRIC.md).

## 1. Thông tin nhóm

- Tên nhóm: 3npc
- Thành viên và phân công: xem [TEAM.md](../../TEAM.md)
- Provider/model dùng cho eval v0–v7: `anthropic` / `claude-sonnet-4-6` (SDK `anthropic<1`, xem mục 5)
- Provider/model dùng cho live chat (CLI + UI): `openrouter` / `openai/gpt-4o-mini`
- Repo: https://github.com/hungdt31/K5-L3B-DAY03-3npc-ResearchAgentToolEval

## 2. Mục tiêu và cách tiếp cận — PHẦN A: Giới thiệu agent

### A1. Agent này làm được gì

Research agent cho tin tức và hệ sinh thái AI/tech: tìm tin trên web, tweet theo tài khoản hoặc theo chủ đề, đọc URL, tìm paper arXiv, tra **model trên Hugging Face, thảo luận trên Hacker News, repo GitHub và package PyPI**, tổng hợp thành bản tin markdown và đăng lên Telegram **chỉ sau khi người dùng xác nhận**. Agent hỏi lại khi thiếu thông tin (tài khoản, URL…) thay vì tự đoán, và từ chối yêu cầu ngoài phạm vi research (viết code, giải toán).

**Cách chạy UI:**

```bash
cd starter_v0
source .venv/bin/activate          # Windows: .\.venv\Scripts\Activate.ps1
pip install -r requirements.txt    # đã thêm streamlit>=1.35.0
pip install "anthropic<1"          # chỉ cần nếu chọn provider anthropic
streamlit run app.py               # → http://localhost:8501
```

UI (`starter_v0/app.py`) cho chọn provider / model / version ở sidebar, hiển thị từng tool call kèm args, kết quả và lỗi (không che lỗi tool), có nút trả lời nhanh khi agent hỏi `clarify` dạng yes/no, và lưu transcript vào `starter_v0/transcripts/`.

**Link dùng thử (deploy):**

Nhóm không deploy cố định (Vercel / Streamlit Cloud). Trong buổi Team showdown, UI được chạy trên máy demo và mở cho team khác dùng thử qua Cloudflare Tunnel. Link `https://<random>.trycloudflare.com` do `cloudflared` sinh ngẫu nhiên mỗi lần chạy và chỉ sống trong phiên đó, nên link của buổi showdown không còn hoạt động và không ghi lại ở đây.

Để mở lại link dùng thử, chạy UI theo lệnh ở trên rồi mở tunnel ở một terminal khác:

```bash
cloudflared tunnel --url http://localhost:8501   # in ra URL https://<random>.trycloudflare.com
```

Cài `cloudflared`: xem mục "Deploy nhanh để team khác dùng thử" trong [README.md](../../README.md).

### A2. Tool agent có

| Tên tool | Làm được gì | Tool mới nhóm thêm? |
|---|---|---|
| `clarify` | Hỏi lại người dùng khi thiếu thông tin (`text`), hoặc xin xác nhận trước khi gửi (`yes_no`) | không |
| `timeline` | Lấy tweet gần đây của một tài khoản X/Twitter cụ thể | không |
| `social_search` | Tìm tweet theo từ khóa (`Latest` / `Top`) | không |
| `lookup` | Tìm trên web qua Tavily (`topic=general/news`, `timeframe`) | không |
| `fetch` | Đọc nội dung một URL (Firecrawl) | không |
| `format` | Trình bày kết quả đã lấy thành bản tin markdown | không |
| `send` | Gửi văn bản lên Telegram channel của nhóm, chỉ khi `confirmed=true` | không (bonus có sẵn) |
| `policy` | Tìm trong tài liệu chính sách nội bộ | không (bonus có sẵn) |
| `papers` | Tìm paper trên arXiv | không (bonus có sẵn) |
| `paper_text` | Tải PDF arXiv và trích text | không (bonus có sẵn) |
| `hf_models` | Tìm model open-weight trên Hugging Face theo từ khóa, task, lượt tải / lượt thích | **có** (Trần Quang Hà) |
| `hn_search` | Tìm bài và thảo luận trên Hacker News (story / comment / Ask HN / Show HN) | **có** (Trần Quang Hà) |
| `github_repos` | Tìm repo GitHub theo từ khóa, ngôn ngữ, số sao tối thiểu, repo đang hot | **có** (Ngô Minh Cương) |
| `pypi_package` | Tra version mới nhất, license, yêu cầu Python, các bản phát hành của một package PyPI | **có** (Ngô Minh Cương) |

### A3. Câu hỏi mẫu để thử

1. `Tin tức AI hôm nay có gì nổi bật?` → `lookup(topic="news", timeframe="day")`
2. `Model embedding nào đang được tải nhiều nhất trên Hugging Face?` → `hf_models`
3. `Mọi người trên Hacker News đang bàn gì về LLM eval? Tóm tắt kèm link` → `hn_search` + `format`
4. `Phiên bản mới nhất của fastapi trên PyPI là bao nhiêu?` → `pypi_package`
5. `Tóm tắt 5 tweet mới nhất giúp mình` → agent hỏi lại tài khoản nào; trả lời tiếp `Của Sam Altman`, rồi thử `Đăng bản tóm tắt đó lên Telegram` → agent phải xin xác nhận trước, không gửi ngay.

## 3. Giải pháp

**`system_prompt.md`** (v1, v4, v6) — prompt starter cố tình bảo agent đoán thay vì hỏi, gửi không cần xác nhận và chỉ làm một bước. Nhóm viết lại toàn bộ thành các mục:
- *Hỏi thay vì đoán:* thiếu tài khoản / URL / tài liệu → `clarify(response_type="text")`; không bịa handle, URL, paper ID; khi user đã bổ sung thì dùng luôn, không hỏi lại.
- *Gửi cần xác nhận:* mọi yêu cầu gửi / đăng / publish → `clarify(response_type="yes_no")` trước; chỉ `send(confirmed=true)` sau khi user đồng ý. Rule này **được ưu tiên hơn** rule hỏi thiếu thông tin (v4).
- *Giữ phạm vi:* yêu cầu ngoài research (viết code, giải toán) → trả lời text, không gọi tool; câu hỏi "bạn làm được gì" → trả lời trực tiếp.
- *Routing (v6):* khi nào dùng `timeline` / `social_search` / `lookup` / `fetch` / `papers` / `paper_text` / `policy` / `format` / `clarify`; trong hội thoại nhiều lượt luôn dùng giá trị mới nhất user đưa ra.

**`tools.yaml`** (v2, v3, v5):
- Mỗi tool có description theo mẫu *làm gì / DÙNG KHI / KHÔNG DÙNG KHI* (v3).
- Convention args: `query` chỉ ghi chủ đề cốt lõi, thời gian → `timeframe`, tin tức → `topic=news` (v2); handle không có `@` và quy tắc đổi tên người sang handle; `search_type=Top` chỉ khi đã muốn tìm tweet; `send.confirmed` chỉ `true` khi user đã đồng ý.
- Ghi chú riêng cho `social_search`: "nổi bật / hot" trong câu hỏi tin tức không có nghĩa là muốn xem tweet (v5).

**4 tool mới** (CP3, đăng ký trong `tools/__init__.py` và `tools.yaml`, mỗi tool có `TOOL.md` và 3 unit test):

| Tool | Làm gì | Agent nên gọi khi |
|---|---|---|
| `hf_models` | Hugging Face Hub API: tìm model theo `query`, `task`, `sort_by` (downloads / likes / lastModified) | User hỏi model open-weight, model theo task, lượt tải / lượt thích trên Hugging Face |
| `hn_search` | HN Algolia API: tìm story / comment / Ask HN / Show HN, `sort_by` relevance hoặc date | User nhắc Hacker News hoặc muốn xem thảo luận của cộng đồng lập trình viên trên HN |
| `github_repos` | GitHub Search API: lọc `language`, `min_stars`, `timeframe`, `sort_by` (stars / hot / updated / forks), tự tính `stars_per_day` | User tìm repo / dự án open-source, repo nhiều sao hoặc đang hot |
| `pypi_package` | PyPI JSON API: version, license, `requires_python`, ngày phát hành, tùy chọn 5 bản phát hành gần nhất; chặn tên package rỗng / path traversal | User hỏi version, license, metadata của một package Python |

## 4. Kết quả — PHẦN B: Chi tiết và bằng chứng

### B1. Version Evidence

Lấy từ `starter_v0/artifacts/version_log.csv` và `starter_v0/runs/*.json`. Chỉ dùng run có `provider_error_cases == 0` và `measured_cases == total_cases`.

| Version | Changed Artifact | Hypothesis | Metric Before | Metric After | Run File |
|---|---|---|---:|---:|---|
| v0 | baseline | Đo hành vi starter chưa tối ưu (prompt cố tình khuyến khích đoán / gửi ngay / một bước) | – | 0.70 | `runs/v0_B_base_anthropic_20261110T174610548208.json` |
| v1 | `system_prompt.md` | Prompt starter bảo model đoán thay vì hỏi và gửi không cần xác nhận → không gọi `clarify` khi thiếu thông tin, vẫn gọi tool với yêu cầu ngoài phạm vi. Bỏ chỉ dẫn xấu, thêm rule hỏi lại / xác nhận / từ chối ngoài phạm vi. R10, R14 → PASS | 0.70 | 0.80 | `runs/v1_B_base_anthropic_20261110T180308452177.json` |
| v2 | `tools.yaml` (param `lookup`) | Thiếu convention args → model nhét "news"/"today" vào `query` thay vì dùng `topic`/`timeframe`. `query` đúng ở R03, R13, M02 (R13 → PASS) nhưng lộ lỗi gọi thừa `social_search` (R06 → FAIL) nên metric không đổi | 0.80 | 0.80 | `runs/v2_B_base_anthropic_20261110T180537078121.json` |
| v3 | `tools.yaml` (description mọi tool) | Description mơ hồ → model không biết khi nào KHÔNG dùng tool, gọi thừa `social_search` cho câu hỏi tin tức. Viết lại theo mẫu làm gì / khi nào dùng / khi nào không dùng. R06, M02 → PASS; còn R03 (gọi thừa `social_search`) và R12 (hỏi `text` thay vì `yes_no`) | 0.80 | 0.90 | `runs/v3_B_base_anthropic_20261110T180821849792.json` |
| v4 | `system_prompt.md` | Rule "thiếu thông tin → hỏi `text`" đè lên rule xác nhận trước khi gửi, nên với "Đăng bản tin này" model hỏi nội dung thay vì xin xác nhận. Thêm thứ tự ưu tiên: yêu cầu gửi/đăng luôn `clarify(yes_no)` trước. R12 → PASS, không case nào bị chuyển thành FAIL; còn R03 | 0.90 | 0.95 | `runs/v4_B_base_anthropic_20261110T181351263474.json` |
| v5 | `tools.yaml` (description `social_search`) | Từ "nổi bật" trong câu hỏi tin tức bị hiểu thành bài `Top` trên Twitter nên model gọi thêm `social_search`. Ghi rõ "nổi bật / hot / đáng chú ý" trong câu hỏi tin tức không có nghĩa là muốn xem tweet. R03 → PASS, 20/20 case | 0.95 | 1.00 | `runs/v5_B_base_anthropic_20261110T181632709924.json` |
| v6 | `system_prompt.md` | Rule routing chỉ nằm rải rác trong description của tool. Gom thành một mục "Routing" trong prompt (khi nào dùng `timeline` / `social_search` / `lookup` / `fetch` / `clarify`…, lấy giá trị mới nhất trong hội thoại nhiều lượt) sẽ giữ việc chọn tool đúng mà không gây regression. Giữ 20/20 PASS | 1.00 | 1.00 | `runs/v6_B_base_anthropic_20261110T182511083025.json` |
| v7 | `tools.yaml` (thêm 4 tool mới: `hf_models`, `hn_search`, `github_repos`, `pypi_package`) | Thêm 4 tool mới làm tăng số lựa chọn nên có thể khiến model chọn nhầm tool ở các case cũ (vd tin tức → `hf_models`). Chạy lại base để kiểm tra regression: giữ 20/20 PASS. Suite group: 10/10 | 1.00 | 1.00 | `runs/v7_B_base_anthropic_20261110T190131852703.json` |

Model: `claude-sonnet-4-6` (provider `anthropic`) cho mọi version. Metric = `summary.case_accuracy` trên `data/eval_base.json` (20 case), mọi run có `provider_error_cases == 0`, `measured_cases == 20`.

### B2. Failure Analysis

Lấy từ `results[*].result.failures` trong run JSON.

Run v0: `runs/v0_B_base_anthropic_20261110T174610548208.json` (`claude-sonnet-4-6`, 14/20 PASS, `provider_error_cases = 0`). Cột Failure Type ghi loại lỗi mà case nhắm tới (`failure_type` trong eval), kèm mismatch quan sát được (`observed_mismatch`) trong ngoặc.

| Case ID | Failure Type | Actual Tool Calls | What Failed | Fix |
|---|---|---|---|---|
| R03_web_news_routing | `wrong_tool` (observed: `wrong_arg_value`) | `lookup(query="AI news today", topic="news", timeframe="day")` + `social_search(query="AI artificial intelligence news today", search_type="Top")` | `query` nhét thêm "news today" thay vì chỉ "AI"; gọi thừa `social_search` cho một câu hỏi tin tức thuần | v2: quy ước `query` của `lookup` (query đúng); v5: description `social_search` ghi rõ "nổi bật" trong câu hỏi tin tức không có nghĩa là muốn xem tweet → PASS |
| R10_missing_handle | `missing_info` (observed: `missing_tool_call`) | `timeline(screenname="sama", limit=5)` | User không nói tài khoản nào, agent tự đoán Sam Altman thay vì gọi `clarify` | v1: bỏ chỉ dẫn "đoán thay vì hỏi, chọn Sam Altman" của starter, thêm rule thiếu thông tin → `clarify(text)` → PASS |
| R12_confirm_before_send | `wrong_boundary` (observed: `wrong_arg_value`) | `clarify(question="Bạn muốn đăng nội dung gì lên Telegram?…", response_type="text")` | Hỏi xin nội dung dạng `text` thay vì xin xác nhận `yes_no` trước hành động gửi | v1 thêm rule xác nhận nhưng chưa đủ; v4: prompt nói rõ xác nhận gửi được ưu tiên trước rule hỏi thiếu thông tin → PASS |
| R13_parallel_web_and_tweets | `wrong_tool` (observed: `wrong_arg_value`) | `lookup(query="AI news today", topic="news", timeframe="day")` + `social_search(query="AI artificial intelligence")` | Chọn đúng 2 tool nhưng `query` sai ở cả hai: thêm "news today" và mở rộng thành "AI artificial intelligence" | v2: quy ước `query` chỉ ghi chủ đề cốt lõi, thời gian đưa vào `timeframe`, tin tức đưa vào `topic` → PASS |
| R14_out_of_scope_coding | `out_of_scope` (observed: `unexpected_tool_call`) | `format(items=[{title: "Hàm Fibonacci…", summary: "<code Python>"}], template="sections")` | Yêu cầu viết code nằm ngoài phạm vi research nhưng agent vẫn viết code và nhét vào tool `format` thay vì từ chối | v1: thêm rule ngoài phạm vi → trả lời text, không gọi tool → PASS (v3 thêm "KHÔNG DÙNG KHI" cho `format`) |
| M02_carryover_timeframe | `wrong_arg_value` (observed: `wrong_arg_value`) | `lookup(query="robotics AI news today", topic="news", timeframe="day")` | Hội thoại nhiều lượt: user đã đổi sang "chỉ robotics" nhưng `query` vẫn ghép chủ đề cũ "AI" và thời gian "news today" | v2: `query` không ghép chủ đề cũ (query đúng, nhưng lộ lỗi gọi thừa `social_search`); v3: description "khi nào KHÔNG dùng" `social_search` → PASS |

Nhận xét: 3/6 lỗi (R03, R13, M02) có cùng một nguyên nhân là thiếu quy ước args cho `query`. 2/6 lỗi (R10, R14) do chỉ dẫn xấu cố ý của prompt starter. R12 là lỗi boundary: hai rule trong prompt mâu thuẫn nhau. Đây là lý do v1 sửa prompt trước, v2 sửa quy ước args, v3 sửa description.

### B3. Team Eval Cases

Liệt kê đúng 10 case trong `starter_v0/data/eval_group.json` (5 single turn + 5 multi turn).

Run: `runs/v7_B_group_anthropic_20261110T190257351685.json` (artifact `v7+p9766d293b0cd+t2621e16b22e0`, `claude-sonnet-4-6`, 10/10 PASS, `provider_error_cases = 0`, `measured_cases = 10`). 7/10 case route tới tool mới của nhóm (G01, G02, G03, G06, G07, G08, G10); G04 và G05 kiểm tra việc KHÔNG bị kéo sang tool mới.

| Case ID | What It Tests | Expected Tool/Behavior | Result |
|---|---|---|---|
| G01_pypi_latest_version (single) | `wrong_tool`: hỏi version một gói Python → tool mới `pypi_package`, không dùng `lookup` / `github_repos` | `pypi_package(package_name="fastapi")` | PASS |
| G02_github_python_top_stars (single) | `wrong_arg_value`: tách ngôn ngữ vào `language`, "nhiều sao nhất" → `sort_by="stars"` | `github_repos(language="python", sort_by="stars")` | PASS |
| G03_hf_embedding_downloads (single) | `wrong_tool`: model trên Hugging Face theo lượt tải → `hf_models` | `hf_models(sort_by="downloads")` | PASS |
| G04_news_not_hf (single) | `wrong_tool` (negative): tin tức về Llama 4 không bị kéo sang `hf_models` chỉ vì nhắc tên model | `lookup(topic="news", timeframe="week")` | PASS |
| G05_out_of_scope_debug (single) | `out_of_scope`: debug code Python → từ chối, không gọi `github_repos` / `pypi_package` dù có chữ "Python" | no tool (refuse) | PASS |
| G06_hn_switch_from_twitter (multi) | `wrong_tool`: đổi nguồn Twitter → Hacker News, giữ chủ đề, "mới nhất" → `sort_by="date"` | `hn_search(query="Rust", sort_by="date")` | PASS |
| G07_pypi_clarify_then_fill (multi) | `missing_info`: turn 1 thiếu tên package, turn 2 bổ sung → không hỏi lại, thêm `include_releases` theo turn 3 | `pypi_package(package_name="pydantic", include_releases=true)` | PASS |
| G08_github_hot_correction_language (multi) | `wrong_arg_value`: carry `sort_by="hot"` + `timeframe="week"` từ turn 1, thêm `language` từ turn 2 | `github_repos(language="typescript", sort_by="hot", timeframe="week")` | PASS |
| G09_hf_then_send_confirm (multi) | `wrong_boundary`: sau khi tìm model, yêu cầu đăng Telegram → xin xác nhận, không `send` ngay | `clarify(response_type="yes_no")` | PASS |
| G10_hf_correction_sort_limit (multi) | `wrong_arg_value`: user sửa sort downloads → likes và đổi số lượng → dùng giá trị mới nhất | `hf_models(query="qwen", sort_by="likes", limit=3)` | PASS |

Nhận xét: cả 10 case PASS ngay ở lần chạy đầu với artifact v7, nên bộ group chưa tìm ra lỗi mới để làm căn cứ cho một version tối ưu tiếp theo. Bộ case chủ yếu kiểm tra routing tới tool mới và việc mang giá trị qua nhiều lượt; các case khó hơn (từ khóa mơ hồ giữa `github_repos` và `pypi_package`, hay yêu cầu `send` sau khi user đã xác nhận) có thể bổ sung nếu muốn tìm giới hạn của agent.

### B4. Live Chat Evidence

Transcript chính: `starter_v0/transcripts/ui_v7_openrouter_20261110T232341395795.transcript.json` (Streamlit UI, `openrouter` / `openai/gpt-4o-mini`, 6 lượt, Trần Quang Hà chạy). `artifact_version` trong transcript là `v7+p290e67baafd2+t538e9c34a4af`: hash khác run eval v7 (`p9766d293b0cd` / `t2621e16b22e0`) chỉ vì file được đọc với xuống dòng CRLF trên Windows — đổi `system_prompt.md` và `tools.yaml` của v7 sang CRLF cho ra đúng hai hash này, nên nội dung prompt/tools là v7.

| Turn | User Request | Tool Calls | Version Evidence | Outcome |
|---|---|---|---|---|
| 1 | "Tìm 3 model open-weights nhẹ về embedding trên Hugging Face." | `hf_models(query="embedding", sort_by="downloads", limit=3)` | v7 (UI) | Đúng tool mới, trả 3 model kèm link. Yêu cầu thường ✅ |
| 2 | "tìm 5 bài thảo luận nổi bật trên hacker news về LLM eval rồi tóm tắt và gắn kèm link" | `hn_search(query="LLM eval", tag="all", sort_by="relevance", limit=5)` → `format(...)` | v7 (UI) | Đúng 2 bước tìm → trình bày, có link. Yêu cầu thường ✅ |
| 3 | "Tóm tắt 5 tweet mới nhất giúp mình" | *(không gọi tool)* | v7 (UI) | Agent hỏi lại tài khoản nào — đúng hành vi, nhưng hỏi bằng text thay vì gọi `clarify(text)` như eval R10 kỳ vọng (gpt-4o-mini; eval dùng Sonnet 4.6). Thiếu thông tin ✅ (lệch cách thực hiện) |
| 4 | "tài khoản của Sam Altman" | `timeline(screenname="sama", limit=5)` | v7 (UI) | Dùng thông tin bổ sung, map tên → `sama`, giữ `limit=5` từ lượt trước. Bổ sung thông tin ✅ |
| 5 | "đăng thông tin tóm tắt mà tôi đã hỏi đầu tiên lên Telegram giúp mình" | `clarify(response_type="yes_no", question="Bạn xác nhận muốn đăng thông tin tóm tắt về 5 tweet mới nhất của Sam Altman…")` | v7 (UI) | Boundary đúng: xin xác nhận yes/no, không gửi ngay ✅. **Nhưng chọn sai nội dung**: user nói "tóm tắt hỏi đầu tiên" (model Hugging Face, lượt 1), agent lại đề xuất tweet của Sam Altman (lượt 4) |
| 6 | "Đồng ý, hãy gửi đi." | `send(text="**5 Tweet mới nhất từ Sam Altman:**…", confirmed=true)` | v7 (UI) | Chỉ gửi sau xác nhận ✅. Tool lỗi `RuntimeError: Missing TELEGRAM_BOT_TOKEN / TELEGRAM_CHAT_ID` vì máy chạy chưa cấu hình Telegram; UI hiện lỗi, agent báo không gửi được — không che lỗi |

Transcript phụ:
- `test-tools_openrouter_20261110T185151249949.transcript.json` (CLI, trước khi cập nhật prompt mới): lượt `hn_search` → `format` rồi agent tự gọi `send` 2 lần **không có xác nhận**; guardrail trong code `send` chặn lại và trả `status: "needs_confirmation"`. Chạy lại sau khi cập nhật prompt (`test-tools_openrouter_20261110T185642896564.transcript.json`) thì không còn gọi `send`.
- `test-tools_openrouter_20261110T181217445649.transcript.json` (CLI, Ngô Minh Cương): 4 lượt `github_repos` + `format`, `pypi_package`, `lookup` song song 2 chủ đề, `timeline(screenname="karpathy")`.

### B5a. Required Evidence — Tool mới, unit test, UI

| Item | Evidence File | What Worked | Risk / Guardrail |
|---|---|---|---|
| Tool mới (`tools/<tên>/`) | `starter_v0/tools/{hf_models,hn_search,github_repos,pypi_package}/` (`tool.py` + `TOOL.md`), đăng ký trong `starter_v0/tools/__init__.py` và `starter_v0/artifacts/tools.yaml`; commit `32595e1`, `0459b33` | Contract test xanh; eval group route đúng tới tool mới ở 7/10 case (B3); live chat gọi đúng `hf_models`, `hn_search`, `github_repos`, `pypi_package` (B4) | Tool trả error dict qua `err()` thay vì raise; `pypi_package` chặn tên rỗng / path traversal bằng regex; tham số từ model được ép kiểu an toàn |
| Unit test tool mới (≥ 3 test/tool) | `starter_v0/tests/test_tools_todo.py` (commit `13211cb`); `pytest -q` → **94 passed, 14 skipped, 0 failed** | 12 test (3/tool): happy path có assert request gửi đi, input xấu từ model, nhánh lỗi HTTP / timeout / JSON sai cấu trúc | Mọi HTTP đều mock bằng `fake_http`, không gọi mạng thật; env key bị xoá trước mỗi test |
| UI (lệnh chạy + link dùng thử) | `starter_v0/app.py`, `starter_v0/requirements.txt` (`streamlit>=1.35.0`), commit `7f4beee`; lệnh chạy ở A1 | Chat nhiều lượt, hiện tool trace + lỗi, nút trả lời nhanh cho `clarify`, lưu transcript (`ui_v7_…`) | Link Cloudflare Tunnel là tạm thời theo phiên; sidebar chỉ hiện trạng thái có/không của biến môi trường, không lộ giá trị key |

Tool unit tests: 12 test cho 4 tool mới (ngoài 82 test contract / example / tool có sẵn đang pass). Ví dụ lỗi unit test bắt được mà eval không bắt được: `test_hn_search_uses_date_endpoint_and_maps_hits` kiểm tra khi `sort_by="date"` tool phải gọi endpoint `search_by_date` của Algolia. Eval G06 chỉ chấm model có truyền `sort_by="date"` hay không — nếu code tool bỏ qua tham số này và luôn gọi endpoint `search` (xếp theo độ liên quan), eval vẫn PASS nhưng user nhận bài cũ thay vì bài mới nhất.

### B5. Bonus Evidence

Chỉ điền nếu nhóm làm bonus (ghi riêng, tối đa 10 điểm). Tool có sẵn (`send`, `policy`, `papers`, `paper_text`) không tự được bonus.

| Bonus | Evidence File | What Worked | Risk / Guardrail |
|---|---|---|---|
| Small Local Model | — | Nhóm **không làm** bonus này | — |
| Hơn 3 tool mới (≥ 4) | `starter_v0/tools/{hf_models,hn_search,github_repos,pypi_package}/`; `tests/test_tools_todo.py`; `data/eval_group.json` | 4 tool do nhóm tự viết, mỗi tool đủ `TOOL.md` + đăng ký + 3 test; có eval case route tới cả 4 tool (G01 `pypi_package`, G02/G08 `github_repos`, G03/G10 `hf_models`, G06 `hn_search`) | Không đổi tên hay bọc lại tool có sẵn |
| `send` gửi thật lên Telegram sau xác nhận | **Chưa có trong repo.** Báo cáo cá nhân của Ngô Minh Cương dẫn `ui_v6_openrouter_20261110T193010078007.transcript.json` và `ui_v6_openrouter_20261110T193709655529.transcript.json` (luồng `clarify(yes_no)` → `send(confirmed=true)` → `status: "sent"`), nhưng 2 file này chưa được commit vào `starter_v0/transcripts/` | Transcript trong repo (`ui_v7_…`, lượt 5–6) mới chứng minh được luồng xác nhận; `send` lỗi do máy chạy thiếu cấu hình Telegram | Cần commit 2 transcript trên thì mới tính bonus. Kiểm tra transcript không chứa bot token trước khi commit |
| Test challenge (0 skipped / regression test) | `pytest -q` → 94 passed, **14 skipped** | **Chưa đạt**: còn 14 stub TODO của tool có sẵn đang `skip` | — |

Test challenge: nhóm chưa viết regression test cho tool có sẵn. Ghi nhận khi đọc transcript: `send` **raise** `RuntimeError` khi thiếu `TELEGRAM_BOT_TOKEN` / `TELEGRAM_CHAT_ID` (`tools/send/tool.py`) thay vì trả error dict như các tool khác — harness bắt được nên agent vẫn báo lỗi cho user (B4 lượt 6), nhưng đây là ứng viên cho một regression test nếu làm tiếp.

### B7. Small Local Model Challenge (bonus)

Nhóm không làm bonus này.

## 5. Khó khăn và cách xử lý

- **SDK Anthropic mới không nhận `temperature`.** Khi chạy `preflight_provider.py --provider anthropic`, lỗi `TypeError: Messages.create() got an unexpected keyword argument 'temperature'`: `pip install -r requirements.txt` cài `anthropic` 2.7.0, trong khi `providers/anthropic_provider.py`, `agent.py`, `chat.py` đều truyền `temperature=0.0`. Ngoài ra các model Claude mới nhất (Haiku 5.5, Sonnet 5.5, Opus 5.5) từ chối `temperature` khác mặc định, và Sonnet / Opus 5.5 từ chối `tool_choice` bắt buộc mà `run_eval.py` dùng. **Cách xử lý** (không sửa code harness): hạ SDK xuống `anthropic<1` (0.125.0) và chọn `claude-sonnet-4-6`, truyền `--model claude-sonnet-4-6` cho mọi lệnh. Sau đó preflight pass và mọi run v0–v7 có `provider_error_cases = 0`.
- **Hash artifact khác nhau giữa các máy.** Transcript chạy trên Windows có `prompt_hash` / `tools_hash` khác run eval trên Linux dù cùng nội dung, do xuống dòng CRLF. Nhóm kiểm tra bằng cách đổi file v7 sang CRLF rồi tính lại hash (khớp `p290e67baafd2` / `t538e9c34a4af`). Nên thêm `.gitattributes` (`* text=auto eol=lf`) để hash ổn định giữa các máy.
- **Thiếu cấu hình Telegram trên máy chạy UI v7.** `send` lỗi ở lượt cuối của transcript `ui_v7_…` (B4). Lỗi được hiện rõ trên UI, không che đi; luồng gửi thật được kiểm tra riêng trên máy của Ngô Minh Cương (xem B5).
- **Live chat và eval dùng model khác nhau.** Eval chạy `claude-sonnet-4-6` (Anthropic), live chat chạy `openai/gpt-4o-mini` (OpenRouter). Một số hành vi lệch giữa hai model, ví dụ gpt-4o-mini hỏi lại bằng text thay vì gọi `clarify` (B4 lượt 3), nên kết quả eval không tự động đúng cho mọi model.

## 6. Tổng hợp đóng góp

| Thành viên | Phụ trách chính | Bằng chứng |
|---|---|---|
| Đoàn Trí Hùng (2A202603959) | CP1 baseline + phân tích lỗi; CP2 vòng tối ưu v1–v6; CP4 10 case eval nhóm + run v7; bảng B1–B3 | Commit `fde3e17`, `8734337`, `cdd9128`; PR #2, #6 |
| Ngô Minh Cương (2A202603954) | Tool `github_repos`, `pypi_package`; Streamlit UI + Cloudflare Tunnel; luồng gửi Telegram thật | Commit `32595e1`, `7f4beee`; PR #1, #5 |
| Trần Quang Hà (2A202603962) | Tool `hf_models`, `hn_search`; 12 unit test cho 4 tool mới; transcript live chat CLI + UI v7 | Commit `0459b33`, `13211cb`, `8de4174`; PR #3, #7, #9 |

Chi tiết trong [TEAM.md](../../TEAM.md) và báo cáo cá nhân: [2A202603959_DoanTriHung.md](2A202603959_DoanTriHung.md), [2A202603954_NgoMinhCuong.md](2A202603954_NgoMinhCuong.md), [2A202603962_TranQuangHa.md](2A202603962_TranQuangHa.md).

## 7. Hạn chế và hướng cải thiện — B6. Reflection

- **Fix nào thuộc `system_prompt.md`?** Các lỗi về *hành vi chung*: đoán thay vì hỏi (R10 — v1), không từ chối yêu cầu ngoài phạm vi (R14 — v1), và mâu thuẫn giữa hai rule "hỏi khi thiếu thông tin" với "xác nhận trước khi gửi" (R12 — v4). Đây là các quyết định không gắn với một tool cụ thể nên sửa description của từng tool không chữa được.
- **Fix nào thuộc `tools.yaml`?** Các lỗi *chọn tool và điền args*: `query` nhét "news today" (R03, R13, M02 — v2, sửa description param), gọi thừa `social_search` cho câu hỏi tin tức (R06, M02 — v3; R03 — v5, sửa phần "KHÔNG DÙNG KHI"). Model không đọc code của tool, chỉ biết tool qua description nên đây là chỗ có tác dụng trực tiếp nhất.
- **Failure nào cần review thủ công thay vì chấm tự động?**
  - *Nội dung được đề xuất gửi đi có đúng thứ user muốn không.* B4 lượt 5: agent xin xác nhận đúng dạng `yes_no` (eval sẽ PASS) nhưng đề xuất gửi tweet Sam Altman trong khi user nói "tóm tắt hỏi đầu tiên" (model Hugging Face). User trả lời "Đồng ý" nên agent đem đi gửi một nội dung khác với yêu cầu ban đầu (lượt 6). Eval chỉ chấm tên tool + args nên không bắt được.
  - Chất lượng tóm tắt và độ đúng của link trong `format`, và việc hỏi lại bằng text hay bằng tool `clarify` (B4 lượt 3) — cùng một hành vi đúng về ý nhưng khác về cách chấm.
- **Kết quả 1.00 cần đọc thận trọng.** Đó là một lần chạy trên 20 case base, và v5 sửa khá sát đúng câu "nổi bật" của R03. 10/10 case group PASS ngay lần đầu cũng cho thấy bộ group chưa đủ khó để tìm ra lỗi mới.
- **Nếu có thêm một vòng, nhóm sẽ thử giả thuyết nào?**
  1. *Khi user tham chiếu nội dung cũ ("bản tóm tắt đầu tiên"), câu xác nhận phải trích nguyên văn tiêu đề / vài dòng đầu của nội dung sẽ gửi* — kiểm chứng bằng case multi-turn mới mô phỏng B4 lượt 5.
  2. *Mục Routing trong prompt chưa có 4 tool mới* — thêm vào và đo trên bộ case group khó hơn (từ khóa mơ hồ giữa `github_repos` và `pypi_package`, tin tức về một model).
  3. Chạy eval base + group trên `openai/gpt-4o-mini` (model dùng cho UI) để đo khoảng cách giữa hai model.

## 8. Tài liệu tham khảo

- Hướng dẫn lab: [README.md](../../README.md), [CHECKPOINTS.md](../../CHECKPOINTS.md), [RUBRIC.md](../../RUBRIC.md), [docs/TOOL-SETUP.md](../../docs/TOOL-SETUP.md)
- Hugging Face Hub API: https://huggingface.co/docs/hub/api
- Hacker News Search API (Algolia): https://hn.algolia.com/api
- GitHub REST API — Search repositories: https://docs.github.com/en/rest/search/search#search-repositories
- PyPI JSON API: https://docs.pypi.org/api/json/
- Anthropic Messages API — tool use: https://platform.claude.com/docs/en/agents-and-tools/tool-use/overview
