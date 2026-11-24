---
name: tech-docs
description: Chuẩn viết và tổ chức tài liệu kỹ thuật của dự án — README, kiến trúc hệ thống, ERD, tài liệu API, đặc tả nghiệp vụ, ADR (quyết định kiến trúc), CHANGELOG, hướng dẫn cài đặt/triển khai. Dùng khi tạo hoặc cập nhật bất kỳ tài liệu nào trong docs/, README hay CHANGELOG, và sau mỗi tính năng/thay đổi API.
---

# Tech Docs — Chuẩn tài liệu kỹ thuật

Nguyên tắc: tài liệu **sống cùng code** (trong repo, cập nhật cùng commit), viết cho người mới vào dự án đọc hiểu được, ngắn gọn, có ví dụ chạy được.

## 1. Cấu trúc thư mục tài liệu

Mục lục và quy ước đầy đủ nằm ở `docs/README.md` (nguồn sự thật). Tóm tắt:

```
/
├── README.md · CHANGELOG.md
└── docs/
    ├── README.md               # Mục lục
    ├── steps/                  # 7 master prompt theo thứ tự thực hiện
    ├── 01-tong-quan/           # 00 overview, 01 requirements, 02 feasibility, 03 research
    │   └── business/           # Đặc tả nghiệp vụ theo tính năng (skill business-flow)
    ├── 02-kien-truc/           # 04 architecture, 05 model, 08 compute, 09 kaggle, 18 ADR
    │   └── architecture/       # Sơ đồ chi tiết (Bước 02)
    ├── 03-du-lieu/             # 06 annotation model, 07 dataset & import/export
    │   └── database/           # Thiết kế database (Bước 03)
    ├── 04-giao-dien/           # 10 UI/UX
    ├── 05-phat-trien/          # 11 environment, 12 codebase, 14 testing
    ├── 06-ke-hoach/            # 13 roadmap, 19 readiness
    │   └── project-management/ # Issue, milestone, traceability (Bước 04)
    └── 07-van-hanh/            # 15 packaging, 16 security & licenses, 17 troubleshooting
```

Không đặt file lẻ ở `docs/` gốc, ngoại trừ `README.md`.

## 2. Khi nào cập nhật tài liệu nào

| Thay đổi | Tài liệu phải cập nhật |
|----------|------------------------|
| Thêm/sửa tính năng, quy tắc nghiệp vụ | `docs/01-tong-quan/01_requirements.md` hoặc `docs/01-tong-quan/business/<feature>.md`, `CHANGELOG.md` |
| Thêm/sửa thành phần, giao thức app ↔ worker | `docs/02-kien-truc/04_system_architecture.md` (+ `architecture/`) |
| Thêm/đổi model, adapter | `docs/02-kien-truc/05_model_and_inference_design.md`, `docs/01-tong-quan/03_research_and_references.md` |
| Đổi schema annotation / database | `docs/03-du-lieu/06_annotation_data_model.md`, `docs/03-du-lieu/database/` |
| Thêm/đổi format import/export | `docs/03-du-lieu/07_dataset_and_import_export.md` |
| Đổi màn hình, phím tắt | `docs/04-giao-dien/10_desktop_ui_ux_specification.md` |
| Thêm dependency, biến môi trường, đổi cách cài | `docs/05-phat-trien/11_environment_and_setup.md`, `.env.example`, `README.md` |
| Đổi license / bảo mật | `docs/07-van-hanh/16_security_privacy_and_licenses.md` |
| Quyết định kiến trúc quan trọng | ADR mới trong `docs/02-kien-truc/18_architecture_decision_records.md` |

## 3. Templates

- README gốc: [templates/README.template.md](templates/README.template.md)
- Tài liệu API 1 module: [templates/api-module.md](templates/api-module.md)
- ADR: [templates/adr.md](templates/adr.md)
- Đặc tả nghiệp vụ: `.claude/skills/business-flow/templates/business-spec.md`

## 4. Kiến trúc (`docs/architecture.md`) gồm

1. Tổng quan: hệ thống làm gì, các thành phần chính.
2. Sơ đồ thành phần (Mermaid `flowchart LR`): Browser → Frontend → Backend API → PostgreSQL / dịch vụ ngoài.
3. ERD (Mermaid `erDiagram`) — cập nhật mỗi khi đổi schema.
4. Luồng xác thực (sequence diagram).
5. Cấu trúc thư mục BE/FE (tóm tắt, link tới skill).
6. Quy ước chung: format lỗi, versioning API, múi giờ, logging.

## 5. CHANGELOG

Theo [Keep a Changelog](https://keepachangelog.com/) + Semantic Versioning:

```markdown
## [Unreleased]
### Added
- API `POST /api/v1/enrollments` đăng ký khóa học (BR-01..BR-04)
### Changed
### Fixed
### Breaking
```

## 6. Phong cách viết

- Tiếng Việt, câu ngắn, chủ động. Thuật ngữ kỹ thuật giữ tiếng Anh khi thông dụng (endpoint, token, migration).
- Mỗi tài liệu mở đầu bằng 1–2 câu "tài liệu này dùng để làm gì".
- Ưu tiên bảng, danh sách, sơ đồ Mermaid, ví dụ request/response thật — hơn là đoạn văn dài.
- Lệnh shell để trong code block, copy-paste chạy được.
- Không lặp nội dung giữa các file — link sang nhau.
- Ghi ngày cập nhật ở đầu tài liệu nghiệp vụ/kiến trúc.

## 7. Comment trong code

- Comment giải thích **tại sao**, không phải **làm gì**.
- Tham chiếu quy tắc nghiệp vụ: `// BR-03: không cho hủy khi khóa học đã bắt đầu`.
- Hàm public của service/hook phức tạp: JSDoc ngắn.
- `TODO:` phải kèm lý do hoặc mã việc.
