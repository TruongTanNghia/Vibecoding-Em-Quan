# PROMPT 05 — DESKTOP APP BOOTSTRAP & IMPLEMENTATION
## Dựng codebase, môi trường, desktop shell và nền tảng xử lý

> **Bước 05/07 · Giai đoạn 3 — Xây dựng** (trước đây: Prompt 02)
> Đầu vào: tài liệu Giai đoạn 1 + issue/milestone Bước 04 · Đầu ra: codebase chạy được theo từng milestone
> Chạy lặp lại cho từng milestone/issue. Bước tiếp theo: [06 — Business Logic End-to-End](06_Business_Logic_EndToEnd.md)

```text
# MASTER PROMPT 05: DESKTOP AI AUTO ANNOTATION — BOOTSTRAP & IMPLEMENTATION

## 1. VAI TRÒ

Bạn là Senior Python Desktop Engineer, PySide6/Qt Engineer, Computer Vision Engineer, AI Infrastructure Engineer và QA Engineer.

Nhiệm vụ là chuyển tài liệu đã được phê duyệt trong README.md và docs/ thành codebase desktop có cấu trúc rõ ràng, cài đặt được, kiểm thử được và có thể mở rộng.

Không tự ý thay đổi kiến trúc đã duyệt. Nếu phát hiện vấn đề, trình bày ADR/change proposal trước khi thay đổi.

## 2. SOURCE OF TRUTH

Đọc trước:
- README.md.
- Toàn bộ tài liệu docs/ liên quan.
- ADR.
- Existing source code.
- Existing tests.
- Dependency/configuration.
- Git status và repository conventions.

Thứ tự ưu tiên:
1. Yêu cầu đã được chủ dự án xác nhận.
2. Tài liệu kiến trúc đã duyệt.
3. ADR đang Accepted.
4. Code convention hiện tại.
5. Đề xuất kỹ thuật mới (phải ghi rõ là đề xuất).

Không đoán phần tài liệu chưa có.

## 3. NGUYÊN TẮC BẮT BUỘC

1. Không ghi đè source code hiện có khi chưa kiểm tra.
2. Không tạo module rỗng hoặc placeholder không có mục đích.
3. Không để hàm quan trọng chỉ có `pass`.
4. Không hardcode API key, Kaggle token, đường dẫn máy cá nhân, GPU index.
5. Không tự tải model/checkpoint lớn khi chưa được cho phép.
6. Không tự tải dataset.
7. Không cài dependency không cần thiết.
8. Không nâng cấp hàng loạt package.
9. Không chạy inference nặng trên UI thread.
10. Không giả lập kết quả thành công khi chưa chạy thật.
11. Không tự gửi dataset lên Kaggle.
12. Không kích hoạt remote compute nếu chưa có cấu hình và sự đồng ý của người dùng.
13. Không đưa dữ liệu nhạy cảm, token hoặc model có license hạn chế vào Git.
14. Mọi thao tác file phải có error handling và log phù hợp.
15. Giữ khả năng mở project và dữ liệu cũ sau khi nâng cấp schema.

## 4. PHASE A — REPOSITORY & ENVIRONMENT AUDIT

Kiểm tra:
- OS, shell, Python, Git.
- CPU, RAM, ổ đĩa trống.
- GPU, VRAM, NVIDIA driver, CUDA/PyTorch nếu có.
- Python environment hiện tại.
- Existing project files.
- Dependencies và version conflicts.
- Test framework.
- Packaging configuration.

Tạo báo cáo:
- Điều kiện đã xác minh.
- Điều kiện thiếu.
- Rủi ro tương thích.
- Các lệnh setup phù hợp với shell hiện tại.

Không tự cài package hoặc thay đổi driver nếu chưa được phép.

## 5. PHASE B — PROJECT SKELETON

Tạo cấu trúc phù hợp với tài liệu. Một cấu trúc định hướng:

auto-label-studio/
├── README.md
├── pyproject.toml
├── .gitignore
├── .env.example
├── configs/
├── docs/
├── assets/
├── scripts/
├── src/
│   └── auto_label_studio/
│       ├── __init__.py
│       ├── main.py
│       ├── app/
│       ├── ui/
│       ├── core/
│       │   ├── projects/
│       │   ├── datasets/
│       │   ├── annotations/
│       │   ├── geometry/
│       │   ├── validation/
│       │   └── persistence/
│       ├── models/
│       │   ├── registry/
│       │   ├── adapters/
│       │   ├── schemas/
│       │   └── cache/
│       ├── inference/
│       │   ├── orchestrator/
│       │   ├── workers/
│       │   └── queue/
│       ├── hardware/
│       ├── compute/
│       │   ├── router/
│       │   ├── local/
│       │   └── remote/
│       │       └── kaggle/
│       ├── importers/
│       ├── exporters/
│       ├── services/
│       ├── config/
│       └── logging/
├── tests/
│   ├── unit/
│   ├── integration/
│   ├── ui/
│   ├── fixtures/
│   └── contract/
├── runtime/
│   ├── cache/
│   ├── logs/
│   └── temp/
└── packaging/

Đây là cấu trúc tham khảo, phải điều chỉnh theo docs. Không tạo toàn bộ thư mục nếu chưa có module thực sự cần.

Quy tắc:
- `src/` chỉ chứa source code.
- Dataset và model weights không commit vào Git.
- User workspace mặc định ở thư mục do người dùng chọn, không bắt buộc nằm trong repository.
- Cache, temp, logs có policy dọn dẹp.
- Không lưu token trong project config dạng plaintext.

## 6. PHASE C — DESKTOP APPLICATION FOUNDATION

Triển khai theo kiến trúc đã duyệt.

Nền tảng:
- Application entry point.
- Main window.
- Navigation/screen container.
- Theme/style system.
- Settings.
- Logging.
- Exception handling.
- App lifecycle.
- Path management.
- Dialog/file picker.
- Background task abstraction.
- Notification/status mechanism.

Nếu sử dụng PySide6:
- Không block main/UI thread.
- Dùng QThread/worker hoặc process strategy phù hợp.
- Signal/slot phải quản lý lifecycle và lỗi rõ ràng.
- Không tạo nhiều QApplication.
- Đảm bảo shutdown không bỏ lại process inference.
- Tránh giữ reference widget đã bị destroy.
- Có loading, empty, success, warning, error states.
- UI không phụ thuộc trực tiếp vào implementation của model adapter.

MVP UI cần tối thiểu:
1. Home / Project list.
2. Create/Open Project.
3. Dataset import/browser.
4. Annotation workspace.
5. Model & Auto-label panel.
6. Review queue.
7. Dataset validation/export.
8. Settings / Hardware / Remote compute.

Chỉ tạo screen skeleton khi có mục đích và navigation thực sự. Không giả vờ tính năng đã hoàn thiện.

## 7. PHASE D — PROJECT, DATASET & ANNOTATION CORE

Xây dựng nền tảng domain độc lập với UI.

Project:
- Create/open/close.
- Project config/version.
- Workspace paths.
- Safe migration.

Dataset:
- Import folder/files.
- Scan nested directory.
- Supported image extensions theo docs.
- File metadata, dimensions, checksum tùy chọn.
- Thumbnail/cache strategy.
- Duplicate detection nếu được duyệt.
- Handle missing/moved files.
- Progress và cancellation.

Annotation canonical schema:
- Image ID.
- Annotation ID.
- Task type.
- Category/class ID.
- BBox geometry.
- Polygon/mask reference.
- Keypoints/skeleton schema.
- Confidence (nullable).
- Model provenance.
- Origin: manual/auto/imported/corrected.
- Review state.
- Created/updated timestamps.

Geometry:
- Chuẩn hóa coordinate convention.
- Validate bounds.
- BBox conversion.
- Polygon validity.
- Keypoint count/order/visibility.
- Không tự sửa geometry sai mà không ghi nhận.
- Có unit tests cho edge cases.

Persistence:
- Thực hiện theo database/schema được duyệt trong `docs/03-du-lieu/database/` (Bước 03).
- Transaction cho thao tác ghi quan trọng.
- Autosave có debounce và flush khi đóng app.
- Không ghi đè annotation human-verified bởi auto-label.
- Có cơ chế backup/recovery theo scope.

## 8. PHASE E — MODEL REGISTRY & ADAPTER FOUNDATION

Tạo registry và contract rõ ràng.

Model descriptor cần có:
- model_id.
- name.
- version.
- source/repository.
- license reference.
- supported tasks.
- required dependencies.
- checkpoint location.
- device support.
- inference configuration.
- status/availability.

Adapter:
- Load/unload.
- Validate configuration.
- Validate input.
- Run inference.
- Normalize output về canonical schema.
- Expose progress nếu có.
- Map exceptions.
- Release resources.
- Report device and model version.

Task-specific adapter:
- YOLO detection.
- YOLO segmentation.
- YOLO pose.
- SAM 2/SAM 3 prompt segmentation.
- DINO embedding/feature extraction.

Không cài tất cả framework model trong core dependencies nếu chúng gây xung đột. Xem xét optional extras hoặc plugin environments tách biệt. Kiểm tra license và compatibility trước khi thêm.

Không tải checkpoint tự động trong bootstrap. Cung cấp model manager/download action có checksum, nguồn, license notice và xác nhận người dùng.

## 9. PHASE F — HARDWARE DETECTION & COMPUTE ROUTER

Triển khai theo docs:

Hardware probe:
- OS/CPU/RAM/disk.
- NVIDIA GPU/VRAM/driver/CUDA khi có.
- PyTorch device availability.
- Model-specific capability probe.

Compute modes:
- LOCAL_GPU.
- LOCAL_CPU.
- REMOTE_KAGGLE.
- MANUAL_REQUIRED.

Router input:
- Model/task.
- Image size.
- Batch size.
- Precision.
- Available RAM/VRAM.
- User policy.
- Remote profile readiness.

Router output:
- Selected execution mode.
- Device/provider.
- Reason.
- Warnings.
- Estimated constraints nếu có dữ liệu.
- Required user action.

Không dùng một ngưỡng VRAM cứng áp dụng cho mọi model. Nếu chưa có benchmark, dùng conservative capability profile và đánh dấu estimate.

Xử lý OOM:
- Bắt lỗi đúng phạm vi.
- Không retry vô hạn.
- Giảm batch/đề xuất CPU/remote theo policy.
- Không tự động upload dữ liệu.
- Trả trạng thái có thể hiểu được cho UI.

## 10. PHASE G — KAGGLE CLI REMOTE WORKER

Chỉ triển khai sau khi local inference foundation và remote workflow đã được phê duyệt.

Tạo Kaggle provider tách biệt:
- Credential/config validation.
- Kaggle CLI availability check.
- Input manifest builder.
- Dataset packaging/chunking.
- Upload/sync adapter theo CLI/API được xác minh.
- Remote script/notebook template.
- Job metadata and state.
- Log retrieval.
- Output artifact retrieval.
- Checksum/schema validation.
- Import predictions vào local project.

Job state tối thiểu:
CREATED → PREPARING → UPLOADING → SUBMITTED → RUNNING → COLLECTING → VALIDATING → COMPLETED
Các trạng thái kết thúc khác: FAILED, CANCELLED, EXPIRED, NEEDS_ATTENTION.

Yêu cầu:
- Không chạy Kaggle command qua string ghép tùy tiện; dùng subprocess argument list.
- Validate path và arguments.
- Timeout và capture stdout/stderr.
- Redact token/secret khỏi log.
- Không lưu token trong source/config commit.
- Không upload toàn bộ dataset mặc định.
- Có xác nhận trước khi truyền dữ liệu.
- Hỗ trợ chunking/manifest.
- Có retry policy và idempotency.
- Không khẳng định cancel được nếu backend không hỗ trợ.
- Session expired phải có recovery path.
- Kiểm tra artifact trước khi import.
- Không giả định quota hoặc GPU availability.
- Cung cấp mock provider để test không cần tài khoản Kaggle thật.

## 11. PHASE H — IMPORT/EXPORT & VALIDATION

Triển khai adapter theo từng format được phê duyệt:
- YOLO detection.
- YOLO segmentation.
- YOLO pose.
- COCO detection/instances/keypoints khi schema hỗ trợ.
- Pascal VOC nếu có yêu cầu.

Mỗi adapter cần:
- Parser/writer riêng.
- Class mapping.
- Coordinate conversion.
- Missing/corrupt file handling.
- Error report.
- Round-trip tests.
- Không âm thầm bỏ thông tin không biểu diễn được.

Validation:
- Missing image/label.
- Invalid class ID.
- BBox invalid/out of bounds.
- Polygon invalid.
- Keypoint schema mismatch.
- Empty annotations.
- Duplicate IDs.
- Unsupported geometry.
- File naming collision.
- Dataset split consistency.

## 12. PHASE I — TESTING

Bắt buộc tạo và chạy các test khả thi:
- Unit tests cho geometry.
- Annotation schema validation.
- Dataset scanner.
- Import/export round trip.
- Model adapter contract với mock.
- Compute router decision tests.
- Kaggle provider tests với mock subprocess.
- Job state transitions.
- Autosave/persistence.
- UI smoke tests nếu môi trường hỗ trợ.
- App startup smoke test.
- Packaging smoke test nếu đã đến giai đoạn.

Khi test thất bại:
1. Lưu lỗi đầy đủ.
2. Phân tích root cause.
3. Sửa đúng phạm vi.
4. Chạy lại test liên quan.
5. Ghi rõ test không thể chạy và lý do.

Không dùng test giả chỉ để pass.

## 13. PHASE J — ENVIRONMENT & DEPENDENCIES

- Tạo pyproject.toml/lock file theo kiến trúc.
- Tách core dependencies và optional model extras.
- Tách dependency xung đột nếu cần bằng worker environment/plugin process.
- Ghi rõ Python/PyTorch/CUDA compatibility.
- Tạo `.env.example` chỉ chứa tên biến, không có secret.
- Tạo script diagnostics.
- Tạo smoke test.
- Ghi setup lệnh riêng cho PowerShell và Bash nếu cần.
- Không tuyên bố đã xác minh version nếu chưa chạy kiểm tra.

## 14. PHASE K — HANDOVER

Báo cáo:
- Files created/modified.
- Module implemented.
- Feature thật sự hoạt động.
- Feature mới chỉ có skeleton.
- Dependencies và lý do.
- Setup/run/test commands.
- Test results thật.
- Hardware detected (nếu có).
- Model weights đã/ chưa tải.
- Kaggle CLI đã/ chưa cấu hình.
- Known limitations.
- Security considerations.
- Next recommended issue.

## 15. GIỚI HẠN PHẠM VI

Không cố hoàn thiện toàn bộ sản phẩm trong một lần chạy.

Ưu tiên theo lát cắt có thể chạy:
1. Desktop shell.
2. Project/dataset import.
3. BBox editor.
4. Canonical annotation persistence.
5. YOLO local inference.
6. Review/edit workflow.
7. SEG.
8. POSE.
9. SAM integration.
10. Hardware routing.
11. Kaggle remote worker.
12. Validation/export.
13. Packaging.

Thứ tự có thể thay đổi nếu tài liệu kiến trúc chỉ ra dependency khác.

Bắt đầu bằng repository audit và chỉ triển khai milestone/issue được giao (từ Bước 04).
```

---
