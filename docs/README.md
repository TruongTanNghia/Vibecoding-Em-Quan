# Tài liệu AutoLabel Studio — Mục lục

Tài liệu được chia thành 7 nhóm chủ đề, cộng thêm thư mục `steps/` chứa các prompt quy trình.
Mỗi file giữ **số thứ tự 00–19** theo Bước 01, để các prompt trong `steps/` tham chiếu ổn định.
Đọc theo thứ tự nhóm **01 → 07**.

```
docs/
├── README.md                 ← bạn đang ở đây
├── steps/                    Lộ trình 7 bước (master prompts) — xem steps/README.md
├── 01-tong-quan/             Sản phẩm, yêu cầu, khả thi, nghiên cứu
├── 02-kien-truc/             Kiến trúc hệ thống, model, compute, remote, ADR
│   └── architecture/         ← Bước 02 sẽ tạo (sơ đồ chi tiết + diagrams/*.mmd)
├── 03-du-lieu/               Annotation data model, dataset, import/export
│   └── database/             ← Bước 03 sẽ tạo (schema, ERD, migration, backup)
├── 04-giao-dien/             Đặc tả UI/UX desktop
├── 05-phat-trien/            Môi trường, cấu trúc code, chiến lược test
├── 06-ke-hoach/              Roadmap, checklist sẵn sàng
│   └── project-management/   ← Bước 04 sẽ tạo (issue, milestone, traceability)
└── 07-van-hanh/              Đóng gói, bảo mật & license, xử lý sự cố
```

## 01 — Tổng quan sản phẩm

| # | Tài liệu | Nội dung |
|---|---|---|
| 00 | [Project Overview](01-tong-quan/00_project_overview.md) | Tầm nhìn, phạm vi, MVP, giả định, **5 câu hỏi mở** |
| 01 | [Requirements](01-tong-quan/01_requirements.md) | User story, FR/NFR, quy tắc nghiệp vụ BR, acceptance AC |
| 02 | [Feasibility Study](01-tong-quan/02_feasibility_study.md) | Đánh giá khả thi, chi phí, rủi ro |
| 03 | [Research & References](01-tong-quan/03_research_and_references.md) | Kết quả xác minh model, framework, Kaggle, kèm nguồn |

## 02 — Kiến trúc

| # | Tài liệu | Nội dung |
|---|---|---|
| 04 | [System Architecture](02-kien-truc/04_system_architecture.md) | Thành phần, tiến trình, 6 sơ đồ chính, job state machine, xử lý lỗi |
| 05 | [Model & Inference Design](02-kien-truc/05_model_and_inference_design.md) | Ma trận task–model, registry, adapter, pipeline |
| 08 | [Hardware & Compute Routing](02-kien-truc/08_hardware_and_compute_routing.md) | Probe, capability profile, router, OOM |
| 09 | [Kaggle Remote Worker](02-kien-truc/09_kaggle_remote_worker.md) | Luồng CLI, consent, giới hạn, bảo mật |
| 18 | [Architecture Decision Records](02-kien-truc/18_architecture_decision_records.md) | ADR-001 → 011 (ADR-010 **Blocked**) |

## 03 — Dữ liệu

| # | Tài liệu | Nội dung |
|---|---|---|
| 06 | [Annotation Data Model](03-du-lieu/06_annotation_data_model.md) | Canonical schema, tọa độ, geometry, provenance, workflow state |
| 07 | [Dataset & Import/Export](03-du-lieu/07_dataset_and_import_export.md) | Import, format YOLO/COCO, validation, round-trip |

## 04 — Giao diện

| # | Tài liệu | Nội dung |
|---|---|---|
| 10 | [Desktop UI/UX Specification](04-giao-dien/10_desktop_ui_ux_specification.md) | Bản đồ màn hình, wireframe, phím tắt, trạng thái, accessibility |

## 05 — Phát triển

| # | Tài liệu | Nội dung |
|---|---|---|
| 11 | [Environment & Setup](05-phat-trien/11_environment_and_setup.md) | Python 3.12, dependency, PyTorch/CUDA/driver, setup |
| 12 | [Codebase Structure](05-phat-trien/12_codebase_structure.md) | uv workspace, module, ranh giới import |
| 14 | [Testing & Quality Strategy](05-phat-trien/14_testing_and_quality_strategy.md) | Kim tự tháp test, quality gate, recovery test |

## 06 — Kế hoạch

| # | Tài liệu | Nội dung |
|---|---|---|
| 13 | [Implementation Roadmap](06-ke-hoach/13_implementation_roadmap.md) | Milestone M0–M10, đường găng, DoD |
| 19 | [Project Readiness Checklist](06-ke-hoach/19_project_readiness_checklist.md) | Trạng thái sẵn sàng theo từng giai đoạn |

## 07 — Vận hành

| # | Tài liệu | Nội dung |
|---|---|---|
| 15 | [Packaging, Release & Operations](07-van-hanh/15_packaging_release_and_operations.md) | Đóng gói, runtime manager, cache, backup, phát hành |
| 16 | [Security, Privacy & Licenses](07-van-hanh/16_security_privacy_and_licenses.md) | Quyền riêng tư, secret, threat model, **license YOLO** |
| 17 | [Troubleshooting](07-van-hanh/17_troubleshooting.md) | Xử lý sự cố theo triệu chứng |

## Quy ước

- Tài liệu mới phải nằm trong đúng nhóm chủ đề. Không đặt file lẻ ở `docs/` gốc, ngoại trừ file README này.
- Đầu ra của các bước tiếp theo nằm trong thư mục con của nhóm tương ứng (`architecture/`, `database/`, `project-management/`), theo sơ đồ ở trên.
- Đặc tả nghiệp vụ theo từng tính năng (skill `business-flow`) đặt ở `01-tong-quan/business/`.
- Nhãn trạng thái dùng chung: **CONFIRMED / ASSUMPTION / PROPOSAL / UNVERIFIED**.
