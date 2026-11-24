# 18 — Architecture Decision Records

> Nhật ký các quyết định kiến trúc. Trạng thái: **Proposed** (chờ duyệt) · **Accepted** · **Blocked** (cần chủ dự án quyết định) · **Superseded**.
> Tất cả ADR hiện ở trạng thái **Proposed**, riêng ADR-010 đang **Blocked**. Cập nhật: 2026-11-24.

| ADR | Tiêu đề | Trạng thái |
|---|---|---|
| 001 | Desktop framework: PySide6 + QGraphicsView | Proposed |
| 002 | Kiến trúc phân tầng ports & adapters, core thuần Python | Proposed |
| 003 | Inference chạy trong worker process riêng, giao thức JSON-lines | Proposed |
| 004 | SQLite (WAL) cho mỗi project + filesystem, single writer | Proposed |
| 005 | Canonical annotation schema, tọa độ pixel XYXY, adapter theo task | Proposed |
| 006 | Cùng một gói worker cho local và Kaggle; Kaggle là provider chạy qua CLI | Proposed |
| 007 | SAM 2 qua Hugging Face `transformers` | Proposed |
| 008 | Python 3.12 | Proposed |
| 009 | uv workspace với 3 package: core / studio / worker | Proposed |
| 010 | Chiến lược license cho YOLO (AGPL-3.0) | **Blocked** |
| 011 | Đóng gói: thin app + Runtime Manager cài torch theo driver | Proposed |

---

## ADR-001 — Desktop framework: PySide6 + QGraphicsView
- **Bối cảnh:** cần UI desktop phức tạp (canvas hiển thị hàng trăm annotation, dock, phím tắt), viết bằng Python, ưu tiên Windows, có thể đóng gói.
- **Phương án:**
  1. **PySide6** (LGPL-3.0, bản chính thức của Qt for Python, 6.12, Python 3.10–3.15).
  2. PyQt6 (GPL/thương mại): API tương tự nhưng license chặt hơn cho closed-source.
  3. Electron/Tauri + Python sidecar: UI web linh hoạt, nhưng có 2 runtime, phải tự xây IPC, và canvas lớn vẫn phải tự làm.
  4. Dear PyGui / Tkinter: thiếu widget và hệ sinh thái cho ứng dụng lớn.
- **Quyết định:** PySide6, canvas dùng `QGraphicsView`/`QGraphicsScene`.
- **Hệ quả:** một ngôn ngữ cho toàn bộ dự án; tuân thủ LGPL (liên kết động, đính kèm notice). Rủi ro hiệu năng canvas → spike ở đầu M3; phương án dự phòng là vẽ tùy biến bằng `QOpenGLWidget` hoặc `QRhi`.

## ADR-002 — Ports & adapters, core thuần Python
- **Bối cảnh:** logic geometry/schema cần chạy ở cả app, worker local và trên Kaggle; cần test được mà không có Qt hay GPU.
- **Quyết định:** tầng `ui → application → domain/core`, infrastructure (persistence, compute, io) cài đặt sau interface. `autolabel_core` không import Qt hay torch. Thực thi bằng import-linter.
- **Hệ quả:** test nhanh, dễ thay adapter. Tốn thêm công viết interface; chấp nhận được với quy mô này.

## ADR-003 — Worker process cho inference
- **Bối cảnh:** inference nặng không được chặn UI; CUDA context và các crash của thư viện native không được kéo sập app; các model có thể cần dependency xung đột nhau.
- **Phương án:** (1) QThread trong cùng tiến trình; (2) `multiprocessing`; (3) **subprocess Python riêng với giao thức JSON-lines qua stdin/stdout**; (4) server HTTP local.
- **Quyết định:** (3). Ảnh được truyền bằng đường dẫn, kết quả batch ghi ra file JSONL; phiên interactive SAM giữ model trong worker.
- **Hệ quả:** cô lập lỗi, hủy chắc chắn, có thể dùng runtime khác. Đổi lại phải tự quản lý vòng đời tiến trình (spawn, health check, shutdown) và có độ trễ IPC nhỏ (cần đo cho SAM interactive).

## ADR-004 — SQLite cho mỗi project + filesystem
- **Bối cảnh:** single-user, local-first, project phải dễ di chuyển; dataset lớn tới khoảng 50k–200k ảnh (ESTIMATE).
- **Phương án:** SQLite; PostgreSQL (cần server); DuckDB (thiên về phân tích); chỉ dùng file JSON.
- **Quyết định:** SQLite ở chế độ WAL, một file cho mỗi project, **chỉ tiến trình chính ghi**. Ảnh không lưu trong DB; mask lớn lưu ra file. Bước 03 sẽ quyết định chi tiết schema và ngưỡng.
- **Hệ quả:** không cần server, dễ backup. Nếu sau này cần multi-user thì phải chuyển sang PostgreSQL (data access layer cần giữ được tính khả chuyển).

## ADR-005 — Canonical annotation schema
- **Quyết định:** tọa độ pixel tuyệt đối theo ảnh đã áp EXIF; bbox dạng XYXY; polygon có lỗ và nhiều mảnh; RLE kiểu COCO cho mask; keypoint visibility 0/1/2; có provenance và review status. Mọi format đi qua adapter có capability matrix và round-trip test. Chi tiết ở [06](../03-du-lieu/06_annotation_data_model.md).
- **Hệ quả:** không bị khóa vào một format; chuyển đổi mất mát được báo cáo rõ ràng.

## ADR-006 — Một gói worker cho local và Kaggle
- **Bối cảnh:** remote inference phải cho cùng kết quả với chạy local, và tránh phải duy trì hai bản code.
- **Quyết định:** `autolabel_worker run --manifest` chạy được ở cả hai nơi. Kaggle provider đóng gói input và wheel của worker, chạy `kaggle kernels push` với `timeout`, rồi lấy output về. Kaggle đứng sau interface `RemoteComputeProvider`.
- **Hệ quả:** kết quả nhất quán, test được bằng mock. Bị giới hạn bởi Kaggle: không có lệnh hủy qua CLI, quota không cố định, và có rủi ro về điều khoản ([09](09_kaggle_remote_worker.md), [16 §7](../07-van-hanh/16_security_privacy_and_licenses.md)).

## ADR-007 — SAM 2 qua `transformers`
- **Bối cảnh:** repo `facebookresearch/sam2` khuyến nghị WSL trên Windows và có CUDA extension tùy chọn; wrapper của Ultralytics kéo theo AGPL.
- **Quyết định:** dùng `Sam2Model`/`Sam2Processor` (Apache 2.0) với checkpoint SAM 2.1.
- **Hệ quả:** chạy native trên Windows, license sạch. Hiệu năng so với repo gốc **cần đo**; giữ phương án dự phòng là repo Meta (trên Linux).

## ADR-008 — Python 3.12
- **Lý do:** đây là phiên bản giao nhau của PyTorch 2.15 (≥ 3.11), SAM 3 (≥ 3.12), PySide6 (3.10–3.15), Kaggle CLI (≥ 3.11) và Nuitka (≤ 3.14). Xem [11 §2](../05-phat-trien/11_environment_and_setup.md).

## ADR-009 — uv workspace với 3 package
- **Quyết định:** `autolabel_core`, `autolabel_studio`, `autolabel_worker`; dùng `uv` để quản lý Python, lock file và runtime. Xem [12](../05-phat-trien/12_codebase_structure.md).
- **Hệ quả:** worker gửi lên Kaggle không kéo theo Qt; dependency được cô lập. Cấu trúc này lệch so với cấu trúc tham khảo của Bước 05, và đã nêu lý do.

## ADR-010 — Chiến lược license cho YOLO — **BLOCKED**
- **Bối cảnh:** `ultralytics` và weights YOLO dùng AGPL-3.0; sản phẩm closed-source cần Enterprise License. Mặc định "All Rights Reserved" của Bước 07 xung đột với điều này.
- **Phương án:** A. Dự án dùng AGPL · B. Mua Enterprise License · C. Thay bằng detector có license phù hợp · D. Chỉ dùng cá nhân/học tập · E. Plugin do người dùng tự cài (không khuyến nghị). Chi tiết ở [16 §6](../07-van-hanh/16_security_privacy_and_licenses.md).
- **Cần:** chủ dự án quyết định mô hình phân phối, cộng với ý kiến pháp lý. **Quyết định này chặn M5.**

## ADR-011 — Thin app + Runtime Manager
- **Bối cảnh:** torch+CUDA nặng nhiều GB; bản wheel phù hợp phụ thuộc driver (torch 2.15 cu13x cần driver ≥ 580 và GPU Turing+); PyInstaller one-file không phù hợp.
- **Quyết định:** app dùng PyInstaller one-folder; worker runtime được tạo bằng `uv` khi chạy lần đầu (sau khi người dùng đồng ý), chọn bản CPU / cu12x / cu13x dựa trên NVML.
- **Phương án thay thế:** bundle sẵn torch-CPU; một bản offline cho mỗi biến thể CUDA.
- **Hệ quả:** gói cài nhỏ, phù hợp với từng máy. Lần đầu cần mạng và thêm thời gian; cần chế độ offline (thư mục wheel) ở V1.

---

### Mẫu ADR mới
Dùng [.claude/skills/tech-docs/templates/adr.md](../../.claude/skills/tech-docs/templates/adr.md). Ở Bước 02 có thể tách mỗi ADR thành file riêng trong `docs/02-kien-truc/architecture/` hoặc `docs/adr/`.
