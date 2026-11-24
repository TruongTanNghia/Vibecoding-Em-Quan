# 14 — Testing & Quality Strategy

> Chiến lược kiểm thử cho mọi tầng: unit, integration, UI, model adapter, geometry, import/export, GPU local, remote worker, hiệu năng và recovery.
> Trạng thái: **PROPOSAL** · Cập nhật: 2026-11-24

## 1. Nguyên tắc

1. **Không viết test giả chỉ để pass.** Một test phải có khả năng fail khi hành vi sai.
2. **CI không cần GPU, không cần tài khoản Kaggle, không cần tải model.** Những phần này được cô lập bằng adapter/provider giả có hành vi deterministic. Test cần phần cứng thật được đánh dấu riêng và chạy thủ công.
3. **Mọi quy tắc nghiệp vụ `BR-xx` và tiêu chí `AC-xx` có test**, và tên test chứa mã đó.
4. Không công bố số liệu hiệu năng nếu chưa đo. Mọi benchmark đều ghi lại phần cứng, dataset và tham số.

## 2. Kim tự tháp test

| Tầng | Công cụ (PROPOSAL) | Phạm vi | Chạy ở |
|---|---|---|---|
| Unit | `pytest`, `hypothesis` (property-based) | core: geometry, schema, state machine, validation, router, mapping | CI mỗi commit |
| Contract | `pytest` | Giao thức app ↔ worker; output của adapter khớp canonical schema; JSON Schema của manifest/prediction | CI |
| Integration | `pytest` + SQLite tạm + worker process thật với adapter `fake` | Import dataset, orchestrator + worker, persistence/autosave, export/import, Kaggle provider (subprocess giả) | CI |
| UI | `pytest-qt`, Qt offscreen | Smoke khởi động, các luồng chính, canvas tool, độ phản hồi của event loop | CI (offscreen) |
| E2E | `pytest-qt` + kịch bản | Tạo project → import → auto-label (fake) → review → validate → export | CI (nightly) |
| Model (thật) | marker `@pytest.mark.model` | YOLO/SAM 2 chạy thật trên ảnh fixture: output hợp lệ, provenance đúng | Thủ công / máy có model |
| GPU | marker `@pytest.mark.gpu` | Probe, smoke test, benchmark, OOM thật | Thủ công trên máy có GPU |
| Remote (thật) | marker `@pytest.mark.kaggle` | Một job nhỏ trên Kaggle | Thủ công, cần chủ dự án cho phép |
| Performance | `pytest-benchmark` + script | Import 50k, chuyển ảnh, canvas 500 annotation, truy vấn review | Thủ công / nightly |

## 3. Trọng tâm theo module

### 3.1 Geometry (yêu cầu coverage ≥ 90%)
- Chuyển đổi XYXY ↔ XYWH ↔ normalized, đi một vòng không sai lệch (property-based).
- EXIF orientation 1–8: biến đổi tọa độ khi ảnh bị xoay hoặc lật.
- Polygon: tự cắt, suy biến, chiều kim đồng hồ, có lỗ, nhiều mảnh, nằm ngoài biên.
- Mask ↔ RLE ↔ polygon: so diện tích với dung sai; mask rỗng; mask chạm biên.
- Keypoints: thiếu điểm, `v=0` với x,y=0, ngoài biên, sai số lượng so với schema.

### 3.2 State machine
- Mọi chuyển trạng thái hợp lệ thành công; mọi chuyển không hợp lệ ném `E-JOB-001`.
- Workflow state của ảnh được suy ra đúng từ tập annotation (bảng quyết định).

### 3.3 Router và OOM
- Bảng kịch bản snapshot phần cứng × policy × job type → mode và lý do mong đợi ([08 §10](../02-kien-truc/08_hardware_and_compute_routing.md)).
- Adapter giả ném OOM ở lần thứ k → kiểm tra batch giảm dần, số lần retry và trạng thái cuối cùng.

### 3.4 Persistence và autosave
- Transaction bị rollback khi lỗi giữa chừng.
- **Kill test:** chạy app trong subprocess, tạo chỉnh sửa, `kill -9` / `TerminateProcess`, mở lại, rồi kiểm tra dữ liệu (AC-07).
- Migration: tạo DB phiên bản cũ từ fixture → nâng cấp → so sánh; migration bị ngắt giữa chừng → khôi phục từ backup.

### 3.5 Import/Export (round-trip)
Theo [07 §10](../03-du-lieu/07_dataset_and_import_export.md). Fixture gồm các dataset YOLO và COCO **nhỏ, tự tạo** (không dùng dataset có bản quyền), có đủ edge case.

### 3.6 Model adapter
- Contract: mọi adapter đều trả `CanonicalPrediction` hợp lệ theo JSON Schema; tọa độ nằm trong biên; provenance có đủ trường.
- Adapter `fake`: deterministic theo seed, dùng cho mọi test không cần model thật.
- Test với model thật (marker `model`): 5 ảnh fixture, kiểm tra cấu trúc output (không so khớp giá trị chính xác vì kết quả phụ thuộc phiên bản).

### 3.7 Kaggle provider
- Subprocess giả lập trả về stdout/stderr/exit code theo kịch bản ([09 §9](../02-kien-truc/09_kaggle_remote_worker.md)).
- Kiểm tra lệnh được gọi bằng argument list, có `timeout`, và log đã được redact.
- `runner.py` chạy local bằng CPU với manifest mẫu → output đúng schema.

### 3.8 UI
- Watchdog event-loop: đo độ trễ giữa các tick của `QTimer` trong lúc chạy job/import giả lập, khẳng định không có tick nào trễ quá 100 ms (NFR-PERF-01).
- Mỗi màn hình có test cho các trạng thái empty/error/loading.
- Shutdown: đóng app khi job đang chạy → không còn tiến trình worker nào (NFR-REL-02).

## 4. Đo hiệu năng (cách đo, không phải chỉ tiêu cam kết)

| Chỉ số | Cách đo |
|---|---|
| Thời gian/ảnh, throughput | Timestamp đầu/cuối mỗi chunk trong worker; loại bỏ warmup |
| Peak RAM / VRAM | `psutil`, `torch.cuda.max_memory_allocated`, NVML |
| Độ phản hồi UI | Watchdog event-loop (p50/p95/max của độ trễ tick) |
| Chuyển ảnh | Từ lúc bấm phím tới lúc frame đầu tiên được render |
| Thời gian truyền (remote) | Upload, queue, chạy và download được tách riêng |
| Import | Số ảnh/giây khi quét và đọc metadata (SSD và HDD tách riêng) |

Báo cáo benchmark phải ghi: CPU, GPU, driver, RAM, loại ổ đĩa, OS, phiên bản app/torch/model, dataset (tổng hợp hay thật) và tham số. Không ngoại suy kết quả nhỏ thành năng lực đảm bảo.

## 5. Dữ liệu test

- Ảnh tổng hợp sinh bằng script (hình học có ground truth biết trước), cùng một số ảnh có license tự do (CC0) được ghi nguồn trong `tests/fixtures/README.md`.
- Không commit dataset lớn. Dataset 50k ảnh cho test hiệu năng được sinh tại chỗ bằng script.

## 6. Quality gates trong CI

| Gate | Điều kiện |
|---|---|
| Lint/format | `ruff check`, `ruff format --check` |
| Type | `mypy` (strict cho core) |
| Ranh giới | `import-linter` |
| Test | unit + contract + integration + UI smoke pass |
| Coverage | core ≥ 90%, toàn dự án ≥ 70% (PROPOSAL) |
| Bảo mật | secret scan (ví dụ `gitleaks`), `pip-audit` cho dependency |
| License | Kiểm tra license của dependency mới so với danh sách cho phép ([16](../07-van-hanh/16_security_privacy_and_licenses.md)) |

## 7. Recovery test

| Kịch bản | Kỳ vọng |
|---|---|
| Kill app khi đang chỉnh sửa | Dữ liệu mất không vượt quá khoảng debounce |
| Kill worker khi đang chạy chunk | Job → INTERRUPTED; chunk → pending; resume không tạo annotation trùng |
| Mất điện mô phỏng trong lúc migration | Khôi phục từ backup, có thông báo rõ ràng |
| Ổ dataset ngoài bị rút | Ảnh → `missing`, app không crash, relink được |
| Artifact remote bị hỏng | Không import; job → NEEDS_ATTENTION |
| Đĩa đầy khi export | Export thất bại sạch sẽ (thư mục tạm bị xóa), không có export dở dang |
