# Team

## Thông tin nhóm

- Tên nhóm: 3npc
- Lớp: L3B
- Repo nhóm: https://github.com/hungdt31/K5-L3B-DAY03-3npc-ResearchAgentToolEval
- Nhánh nộp: `main`

## Thành viên

| STT | Họ tên         | MSSV        | Email                        | GitHub                                              | Vai trò |
| --: | -------------- | ----------- | ---------------------------- | --------------------------------------------------- | ------- |
|   1 | Đoàn Trí Hùng  | 2A202603959 | koikoidth12@gmail.com        | [hungdt31](https://github.com/hungdt31)             | Nhóm trưởng; baseline, tối ưu prompt / tool declaration v0–v7, eval nhóm, báo cáo B1–B3 |
|   2 | Ngô Minh Cương | 2A202603954 | 26ai.cuongnm5@vinuni.edu.vn  | [ngominhcuong](https://github.com/ngominhcuong)     | Tool `github_repos`, `pypi_package`; Streamlit UI + deploy; luồng gửi Telegram |
|   3 | Trần Quang Hà  | 2A202603962 | 26ai.hatq@vinuni.edu.vn      | [tranha1764](https://github.com/tranha1764)         | Tool `hf_models`, `hn_search`; unit test cho 4 tool mới; live chat transcript |

## Cá nhân

Mỗi thành viên có báo cáo riêng trong `report/team/`:

- [2A202603959_DoanTriHung.md](report/team/2A202603959_DoanTriHung.md)
- [2A202603954_NgoMinhCuong.md](report/team/2A202603954_NgoMinhCuong.md)
- [2A202603962_TranQuangHa.md](report/team/2A202603962_TranQuangHa.md)

## DoanTriHung-2A202603959

### Phần việc phụ trách

- CP1: setup provider Anthropic, chạy preflight / `pytest` / baseline v0, phân tích 6 case FAIL (bảng B2).
- CP2: vòng lặp tối ưu `system_prompt.md` và `tools.yaml` từ v1 đến v6, ghi `version_log.csv` (bảng B1).
- CP4: 10 case trong `data/eval_group.json`, chạy lại base sau khi thêm tool mới (v7) và suite group (bảng B3).
- Review và merge PR #2, #6, #9, #10, #11.

### Đóng góp cụ thể

- Xử lý lỗi SDK `anthropic` 2.x không nhận `temperature` bằng cách hạ xuống `anthropic<1` và chọn `claude-sonnet-4-6`, không sửa code harness.
- Tăng `case_accuracy` trên base từ 0.70 (v0) lên 1.00 (v5), giữ 1.00 ở v6 (thêm rule routing) và v7 (thêm 4 tool mới).
- Viết 10 case eval nhóm (5 single + 5 multi turn), 7 case route tới tool mới; 10/10 PASS.

### Bằng chứng

- Commit `fde3e17` (cp1), `8734337` (cp2), `cdd9128` (cp4); PR #2, #6.
- `starter_v0/runs/v0_…` đến `v7_B_base_…`, `v7_B_group_…`; `starter_v0/artifacts/version_log.csv`.

## NgoMinhCuong-2A202603954

### Phần việc phụ trách

- CP3: 2 tool mới `github_repos` và `pypi_package`.
- CP5: Streamlit UI (`starter_v0/app.py`), link dùng thử qua Cloudflare Tunnel, luồng gửi Telegram thật sau xác nhận.
- Review và merge PR #1, #3, #4, #5, #7, #8.

### Đóng góp cụ thể

- `github_repos`: GitHub Search API, lọc `language` / `min_stars` / `timeframe`, `sort_by` stars / hot / updated / forks, tự tính `stars_per_day`.
- `pypi_package`: PyPI JSON API, version / license / `requires_python` / các bản phát hành gần nhất, chặn tên package không hợp lệ.
- UI hiện tool trace + lỗi, nút trả lời nhanh cho `clarify`, lưu transcript; thêm `streamlit>=1.35.0` vào `requirements.txt`.

### Bằng chứng

- Commit `32595e1` (PR #1), `7f4beee` (PR #5), `12591da` (PR #8).
- `starter_v0/transcripts/test-tools_openrouter_20261110T181217445649.transcript.json`, `ui_v6_openrouter_20261110T185326916931.transcript.json`.
- Transcript gửi Telegram thật (`ui_v6_openrouter_20261110T193010078007`, `ui_v6_openrouter_20261110T193709655529`): **chưa có trong repo, cần commit bổ sung**.

## TranQuangHa-2A202603962

### Phần việc phụ trách

- CP3: 2 tool mới `hf_models` và `hn_search`.
- CP3: unit test offline cho cả 4 tool mới của nhóm.
- CP5: live chat qua CLI và Streamlit UI (v7), tạo transcript minh chứng.

### Đóng góp cụ thể

- `hf_models`: Hugging Face Hub API theo `query` / `task` / `sort_by` (downloads, likes, lastModified) / `limit`.
- `hn_search`: HN Algolia API theo `tag` / `sort_by` (relevance, date), làm sạch HTML trong trích đoạn, trả error dict khi lỗi mạng / JSON.
- 12 unit test (3/tool cho `hf_models`, `hn_search`, `github_repos`, `pypi_package`): happy path, input xấu từ model, nhánh lỗi; `pytest` 94 passed, 0 failed.
- Transcript UI v7 6 lượt: yêu cầu thường với tool mới, thiếu thông tin rồi bổ sung, xin xác nhận trước khi gửi.

### Bằng chứng

- Commit `0459b33` (PR #3), `13211cb` (PR #7), `8de4174` (PR #9).
- `starter_v0/tests/test_tools_todo.py`; `starter_v0/transcripts/test-tools_openrouter_20261110T185151249949.transcript.json`, `test-tools_openrouter_20261110T185642896564.transcript.json`, `ui_v7_openrouter_20261110T232341395795.transcript.json`.
