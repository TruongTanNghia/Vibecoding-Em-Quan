# 19 — Project Readiness Checklist

> Theo dõi mức độ sẵn sàng của dự án theo từng giai đoạn. Trạng thái: **NOT STARTED** · **IN PROGRESS** · **BLOCKED** · **DONE** · **N/A**.
> Cập nhật: 2026-11-24 (kết thúc Bước 01).

## 1. Giai đoạn thiết kế (M0)

| Hạng mục | Trạng thái | Ghi chú |
|---|---|---|
| Bộ tài liệu 00–19 + README | DONE | Bản Draft/Proposal, chờ duyệt |
| Xác minh công nghệ từ nguồn chính thức | DONE | Còn các mục UNVERIFIED ở [03 §6](../01-tong-quan/03_research_and_references.md) |
| Chủ dự án trả lời 5 câu hỏi mở ([00 §8](../01-tong-quan/00_project_overview.md)) | NOT STARTED | |
| Duyệt phạm vi MVP / non-goals | NOT STARTED | |
| Duyệt ADR-001 → 009, 011 | NOT STARTED | |
| **ADR-010 License YOLO** | **BLOCKED** | Cần quyết định mô hình phân phối + ý kiến pháp lý |
| Chấp nhận rủi ro điều khoản Kaggle | BLOCKED | Cần chủ dự án quyết định ([16 §7](../07-van-hanh/16_security_privacy_and_licenses.md)) |
| Bước 02 — sơ đồ kiến trúc chi tiết (`docs/02-kien-truc/architecture/`) | NOT STARTED | |
| Bước 03 — thiết kế database (`docs/03-du-lieu/database/`) | NOT STARTED | Cần có trước M2 |
| Bước 04 — Epic, milestone, issue | NOT STARTED | |

## 2. Môi trường và hạ tầng

| Hạng mục | Trạng thái |
|---|---|
| Git repository + nhánh main | NOT STARTED (thư mục hiện chưa là git repo) |
| uv workspace, lock file | NOT STARTED |
| CI: lint, type, import-linter, test, secret scan | NOT STARTED |
| Máy test có GPU NVIDIA Turing+ | NOT STARTED (chưa biết phần cứng) |
| Máy test không có GPU | NOT STARTED |
| Tài khoản Kaggle để test (đã xác minh số điện thoại) | NOT STARTED |
| Quyền truy cập Hugging Face cho SAM 3 / DINOv3 (V1) | N/A cho MVP |

## 3. Chức năng MVP

| Milestone | Trạng thái |
|---|---|
| M1 Desktop shell | NOT STARTED |
| M2 Project/dataset | NOT STARTED |
| M3 Annotation core + BBox | NOT STARTED |
| M4 SEG + POSE | NOT STARTED |
| M5 Model registry + YOLO local | BLOCKED (ADR-010) |
| M6 SAM 2 | NOT STARTED |
| M7 Hardware + router | NOT STARTED |
| M8 Kaggle remote | BLOCKED (điều khoản Kaggle) |
| M9 Review, validation, import/export | NOT STARTED |
| M10 Performance, recovery, packaging | NOT STARTED |

## 4. Chất lượng trước khi phát hành MVP

| Hạng mục | Trạng thái |
|---|---|
| AC-01 → AC-08 pass, có bằng chứng | NOT STARTED |
| Round-trip test cho mọi adapter import/export | NOT STARTED |
| Kill test / recovery test | NOT STARTED |
| Benchmark có báo cáo (không dùng số bịa) | NOT STARTED |
| Kiểm tra không có kết nối mạng ngầm | NOT STARTED |
| Secret scan sạch; `.gitignore` đúng | NOT STARTED |
| THIRD_PARTY_NOTICES + LICENSE | NOT STARTED |
| Smoke test bản đóng gói trên 3 cấu hình máy | NOT STARTED |
| README cập nhật theo trạng thái thật (Bước 07) | NOT STARTED |
