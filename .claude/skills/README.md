# Bộ Skills của dự án

Mỗi thư mục con là một skill, gồm `SKILL.md` (hướng dẫn chính) và có thể có `templates/` (mẫu dùng lại).
Claude Code tự phát hiện các skill trong `.claude/skills/`, và `CLAUDE.md` bắt buộc đọc chúng trước khi code.

| Skill            | Mục đích                                                                 |
|------------------|--------------------------------------------------------------------------|
| `project-workflow` | **Điều phối** — quy trình end-to-end từ yêu cầu → nghiệp vụ → BE → FE → docs → kiểm tra |
| `business-flow`  | Phân tích yêu cầu, use case, luồng nghiệp vụ, state machine, quy tắc nghiệp vụ |
| `backend-api`    | Kiến trúc backend, CSDL, API REST, validate, auth, xử lý lỗi, test       |
| `frontend-ui`    | Design system, cấu trúc FE, component, state, gọi API, form, responsive  |
| `tech-docs`      | Chuẩn tài liệu kỹ thuật: README, kiến trúc, API, ADR, changelog          |
| `quality-gate`   | Checklist review & kiểm thử trước khi báo hoàn thành                     |

## Cách gọi thủ công

Gõ `/project-workflow`, `/backend-api`, `/frontend-ui`, `/business-flow`, `/tech-docs`, `/quality-gate` trong Claude Code.

## Cập nhật skill

Khi dự án hình thành quy ước mới (đổi stack, thêm pattern), hãy sửa skill tương ứng — skill là "nguồn sự thật" về cách làm của dự án.
