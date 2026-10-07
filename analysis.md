# Bài làm Block 1 — Tra cứu chính sách đúng phiên bản

## Vị trí bài làm

Theo yêu cầu người dùng, sửa trực tiếp project này, không dùng bản nộp tách riêng.

- `stage-01-files/tools/files.py`, `stage-02-skills/tools/files.py`: thêm `list_files`, dùng chung `_resolve` và `_error`.
- `tools/__init__.py` và `agent.py` ở hai stage: export và đăng ký `list_files` trong schema gửi model.
- `workspace/data/policies/` ở stage 01 và 02: hai phiên bản chính sách.
- `stage-02-skills/workspace/skills/refund-policy/`: skill và reference mẫu trả lời.
- Fixtures có bản sao chính sách và skill để reset workspace không xóa bài làm.
- `run_block1.py`: chạy tool thật, model thật, cuộc trò chuyện mới cho từng ca; đổi tên file, chạy lại A/B, khôi phục tên.

## Kết quả và trạng thái xác minh

Mã xử lý filesystem tương đương đã được kiểm tra trực tiếp trên bản sao trước đó: thư mục hợp lệ trả danh sách sắp xếp, file trả `NOT_A_DIRECTORY`, đường dẫn thiếu trả `DIRECTORY_NOT_FOUND`, đường dẫn thoát workspace và tuyệt đối trả `PATH_OUTSIDE_WORKSPACE`; không liệt kê đệ quy. Kiểm tra symlink bị Windows từ chối tạo (`WinError 1`), nên chưa xác minh bằng symlink thực tế.

Sau khi đưa thay đổi vào project hiện tại, công cụ chạy lệnh vẫn lỗi khởi tạo (`helper_unknown_error`, Node `0xc0000142`). Vì vậy chưa xác nhận lại pytest hoặc trace model thật trong project này. Không coi kết quả kỳ vọng dưới đây là kết quả thực tế.

Khi chạy `run_block1.py`, bằng chứng được lưu ngay trong project:

- `stage-01-files/tool-check-results.json`, `stage-02-skills/tool-check-results.json`: kết quả tool thật.
- `<stage>/traces/<case>-index.json`: tên trace, event đọc tài liệu và event trả câu trả lời.
- `block1-trace-results.md`: câu trả lời thực tế và vị trí event trong trace.

## Các trường hợp cần đối chiếu

| Trường hợp | Kết quả cần đạt | Bằng chứng |
|---|---|---|
| Stage 00, A | Không có tool đọc/tìm tài liệu; không đủ căn cứ nội bộ để kết luận | Snapshot tools rỗng, không có tool result đọc chính sách |
| A | Chính sách trước tháng 10; 8 ngày; không đủ điều kiện | `A-index.json`: tìm file, đọc chính sách cũ, dẫn đúng nguồn |
| B | Chính sách từ tháng 10; 10 ngày; đủ điều kiện, không phí | `B-index.json`: stage 02 đọc SKILL.md, reference, chính sách mới; các nội dung này có trong model context |
| A sau đổi tên | Kết luận như A, nguồn `terms-legacy.md` | `A-renamed-index.json`: tìm và đọc tên mới trong lịch sử mới |
| B sau đổi tên | Kết luận như B, nguồn `terms-current.md` | `B-renamed-index.json`: đọc skill/reference và tên mới |
| Thiếu thông tin | Hỏi trạng thái kích hoạt, chưa kết luận | `missing-index.json`: câu trả lời cuối |

## Giới hạn stage 00

Stage 00 không có tool hoặc skill catalog. Agent thiếu khả năng tìm và đọc chính sách nội bộ, thiếu nội dung và phạm vi hiệu lực để áp dụng. Câu trả lời dựa trên kiến thức sẵn có không phải bằng chứng đọc tài liệu; phải kiểm tra tool result thật trong trace.

## Trả lời câu hỏi cuối bài

Tool cung cấp khả năng truy cập dữ liệu: liệt kê thư mục để tìm tên tài liệu hiện có, rồi đọc nội dung thật. Skill hướng dẫn quyết định: chọn phiên bản theo ngày mua, tính ngày lịch theo ngày yêu cầu trong câu hỏi, kiểm tra kích hoạt và đưa kết luận có nguồn.

Chỉ sửa prompt không tạo khả năng tìm file. Tên file cố định sẽ hỏng khi đổi tên; đưa nội dung chính sách vào prompt vi phạm yêu cầu và không chứng minh đã đọc tài liệu hiện có. Cần tool để tìm/đọc và skill để áp dụng đúng.

## Chạy trực tiếp

Tại thư mục chứa README.md và các stage:

```powershell
uv sync --all-packages --locked
uv run --project stage-02-skills python -X utf8 run_block1.py stage-00-chat
uv run --project stage-02-skills python -X utf8 run_block1.py stage-01-files
uv run --project stage-02-skills python -X utf8 run_block1.py stage-02-skills
```

Script tự đọc `.env` tại thư mục cha (`agent-tools-skills-lab/.env`) mà người dùng đã thêm. Có thể truyền `--env` để dùng đường dẫn khác. Không sao chép `.env` hoặc API key vào trace hay bài nộp.
