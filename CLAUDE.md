# CLAUDE.md — Quy tắc làm việc của dự án

## ⚠️ QUY TẮC BẮT BUỘC: ĐỌC SKILLS TRƯỚC KHI CODE

Trước khi bắt đầu **bất kỳ** công việc code, thiết kế, phân tích nghiệp vụ hay viết tài liệu nào, PHẢI:

1. Đọc skill điều phối: `.claude/skills/project-workflow/SKILL.md` (luôn luôn đọc đầu tiên).
2. Xác định loại công việc và đọc skill chuyên môn tương ứng:

| Loại công việc                                   | Skill cần đọc                                  |
|--------------------------------------------------|------------------------------------------------|
| Phân tích yêu cầu, luồng nghiệp vụ, quy tắc      | `.claude/skills/business-flow/SKILL.md`        |
| Thiết kế CSDL, API, service, auth, backend       | `.claude/skills/backend-api/SKILL.md`          |
| Giao diện, component, trang, UX, gọi API ở FE     | `.claude/skills/frontend-ui/SKILL.md`          |
| README, tài liệu API, kiến trúc, ADR, changelog  | `.claude/skills/tech-docs/SKILL.md`            |
| Review, kiểm thử, trước khi báo "xong"           | `.claude/skills/quality-gate/SKILL.md`         |

3. Một tính năng full-stack → đọc **tất cả** skill liên quan theo thứ tự:
   `business-flow` → `backend-api` → `frontend-ui` → `tech-docs` → `quality-gate`.
4. Ở đầu câu trả lời, ghi ngắn gọn: `Skills đã đọc: ...` để người dùng biết.

Nếu yêu cầu của người dùng mâu thuẫn với skill → làm theo người dùng, và đề xuất cập nhật skill.

## Dự án & lộ trình

**AutoLabel Studio** là ứng dụng desktop, local-first, dùng để gán nhãn dữ liệu Computer Vision (BBox / SEG / POSE) với YOLO, SAM 2/3 và DINOv2/v3. Khi máy local không đủ GPU có thể chạy remote qua Kaggle CLI.

Dự án đi theo 7 bước trong [docs/steps/README.md](docs/steps/README.md). Trước khi làm, phải xác định công việc thuộc **bước nào** và đọc prompt của bước đó. Không nhảy bước khi bước trước chưa qua cổng duyệt.

## Tech stack (định hướng theo docs/steps, chốt ở Bước 01 và 03)

- **Ngôn ngữ:** Python
- **Desktop UI:** PySide6/Qt (đề xuất, chốt sau khi phân tích ở Bước 01)
- **Lõi xử lý:** domain services + model adapters (YOLO, SAM, DINO) + compute router (local CPU/GPU ↔ Kaggle CLI)
- **Lưu trữ:** SQLite + filesystem (đề xuất, chốt ở Bước 03)
- **Tài liệu:** Markdown trong `docs/`, sơ đồ bằng Mermaid

> ⚠️ Skill `backend-api` và `frontend-ui` hiện được viết cho web stack (Node/React). Với dự án này, hãy chỉ áp dụng **nguyên tắc** của chúng (phân lớp, validate, xử lý lỗi, đủ 4 trạng thái UI...) và dùng tech stack ở trên. Hai skill này cần được viết lại cho Python desktop.

## Cấu trúc thư mục dự án

```
/
├── CLAUDE.md
├── .claude/skills/      # Bộ skills — nguồn quy chuẩn của dự án
├── docs/                # Mục lục: docs/README.md — tài liệu mới phải vào đúng nhóm
│   ├── steps/           # 7 master prompt theo thứ tự thực hiện
│   ├── 01-tong-quan/    # overview, requirements, feasibility, research (+ business/)
│   ├── 02-kien-truc/    # architecture, model, compute, kaggle, ADR (+ architecture/)
│   ├── 03-du-lieu/      # annotation model, dataset, import/export (+ database/)
│   ├── 04-giao-dien/    # UI/UX
│   ├── 05-phat-trien/   # environment, codebase, testing
│   ├── 06-ke-hoach/     # roadmap, readiness (+ project-management/)
│   └── 07-van-hanh/     # packaging, security & licenses, troubleshooting
└── packages/{core,app,worker}/  # Source code — uv workspace (docs/05-phat-trien/12_codebase_structure.md, ADR-009)
```

## Ngôn ngữ

- Giao tiếp, tài liệu, comment nghiệp vụ: **Tiếng Việt**.
- Tên biến, hàm, file, bảng, API endpoint: **Tiếng Anh**.
