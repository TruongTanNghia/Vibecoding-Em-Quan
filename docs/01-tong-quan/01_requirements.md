# 01 — Requirements

> Đặc tả yêu cầu chức năng, phi chức năng, quy tắc nghiệp vụ và tiêu chí nghiệm thu.
> Trạng thái: **Draft** · Cập nhật: 2026-11-24 · Mã yêu cầu là **provisional** cho đến khi chủ dự án duyệt.

Ưu tiên theo **MoSCoW**: **M** = Must, **S** = Should, **C** = Could, **W** = Won't (trong phiên bản này). Cột "Tier" cho biết yêu cầu thuộc MVP, V1 hay Future, theo [00](00_project_overview.md).

## 1. User stories

| Mã | Persona | Câu chuyện | Tier |
|---|---|---|---|
| US-01 | P1 | Là CV engineer, tôi muốn tạo project và trỏ tới thư mục ảnh có sẵn mà **không phải copy dữ liệu**, để tiết kiệm ổ đĩa. | MVP |
| US-02 | P1 | Tôi muốn chọn task (BBox/SEG/POSE), model và checkpoint rồi chạy auto-label cho cả batch ảnh, để có nhãn nháp nhanh. | MVP |
| US-03 | P2 | Tôi muốn chỉnh sửa box/polygon/keypoint bằng chuột và phím tắt, có undo/redo, để sửa nhãn nhanh. | MVP |
| US-04 | P2 | Tôi muốn có review queue lọc theo trạng thái, confidence và class, để duyệt nhãn có hệ thống. | MVP |
| US-05 | P1 | Tôi muốn ứng dụng tự kiểm tra GPU/VRAM và giải thích vì sao chạy local hay remote, để tránh lỗi OOM bất ngờ. | MVP |
| US-06 | P3 | Máy tôi không có GPU. Tôi muốn gửi job inference lên Kaggle GPU bằng tài khoản của mình mà không cần mở trình duyệt, để vẫn dùng được model lớn. | MVP |
| US-07 | P1 | Tôi muốn export ra YOLO hoặc COCO và import ngược lại không mất thông tin, để dùng ngay với pipeline huấn luyện. | MVP |
| US-08 | P2 | Nếu app crash, tôi muốn mở lại mà không mất phần đã chỉnh sửa. | MVP |
| US-09 | P1 | Tôi muốn chạy YOLO detect rồi dùng SAM 2 tinh chỉnh mask từ box, để có SEG chất lượng cao hơn. | MVP |
| US-10 | P1 | Tôi muốn validation báo lỗi geometry, class và keypoint trước khi export, để dataset sạch. | MVP |
| US-11 | P1 | Tôi muốn gõ tên khái niệm (ví dụ "helmet") để segment mọi instance, để gán nhãn class mới mà không có detector riêng. | V1 (SAM 3, tùy license) |
| US-12 | P1 | Tôi muốn tìm ảnh trùng hoặc gần trùng và gom cụm ảnh tương tự, để làm sạch dataset. | V1 (DINOv2) |

## 2. Yêu cầu chức năng (FR)

### 2.1 Project & Workspace — `FR-PRJ`

| Mã | Yêu cầu | Ưu tiên | Tier |
|---|---|:-:|:-:|
| FR-PRJ-01 | Tạo project với tên, thư mục workspace, task chính và danh sách class ban đầu | M | MVP |
| FR-PRJ-02 | Mở, đóng project; danh sách project gần đây | M | MVP |
| FR-PRJ-03 | Đổi tên, archive project; xóa project phải xác nhận 2 bước và **không xóa ảnh nguồn** | S | MVP |
| FR-PRJ-04 | Project lưu phiên bản schema; tự migrate có backup khi mở bằng app mới hơn | M | MVP |
| FR-PRJ-05 | Cấu hình đường dẫn: workspace, model cache, export mặc định | M | MVP |
| FR-PRJ-06 | Export/import project bundle (metadata + annotation, không kèm ảnh) để backup hoặc chuyển máy | S | V1 |

### 2.2 Dataset — `FR-DS`

| Mã | Yêu cầu | Ưu tiên | Tier |
|---|---|:-:|:-:|
| FR-DS-01 | Import thư mục (quét đệ quy) hoặc chọn file lẻ; định dạng theo [07](../03-du-lieu/07_dataset_and_import_export.md) | M | MVP |
| FR-DS-02 | Lưu tham chiếu (đường dẫn tương đối với dataset root), kích thước, định dạng, dung lượng, mtime; checksum là tùy chọn | M | MVP |
| FR-DS-03 | Import tăng dần: quét lại không tạo bản ghi trùng; phát hiện file mới, file mất, file đổi | M | MVP |
| FR-DS-04 | Báo cáo import: số file đã nhận, bỏ qua, lỗi (kèm lý do) | M | MVP |
| FR-DS-05 | Thumbnail cache trên đĩa, tạo nền, không chặn UI | M | MVP |
| FR-DS-06 | Trình duyệt ảnh dạng lưới có phân trang/virtualization; lọc theo trạng thái, class, lỗi, confidence | M | MVP |
| FR-DS-07 | Phát hiện file thiếu/di chuyển, cho phép relink dataset root | M | MVP |
| FR-DS-08 | Phát hiện trùng lặp chính xác bằng hash | S | V1 |
| FR-DS-09 | Thống kê: số ảnh, số ảnh đã gán, số nhãn theo class, review coverage | M | MVP |
| FR-DS-10 | Quản lý split train/val/test | S | V1 |

### 2.3 Class & Keypoint schema — `FR-CLS`

| Mã | Yêu cầu | Ưu tiên | Tier |
|---|---|:-:|:-:|
| FR-CLS-01 | Tạo, đổi tên, đổi màu, sắp xếp class; ID nội bộ ổn định, không phụ thuộc thứ tự | M | MVP |
| FR-CLS-02 | Xóa class đang được dùng phải chọn: chặn / gộp vào class khác / xóa annotation, có xác nhận | M | MVP |
| FR-CLS-03 | Keypoint schema: tên điểm, thứ tự, skeleton edges, flip pairs; có preset COCO-17 | M | MVP |
| FR-CLS-04 | Mapping class model → class project (ví dụ class `person` của COCO → `worker`) | M | MVP |

### 2.4 Annotation Workspace — `FR-ANN`, `FR-BBX`, `FR-SEG`, `FR-POS`

| Mã | Yêu cầu | Ưu tiên | Tier |
|---|---|:-:|:-:|
| FR-ANN-01 | Canvas zoom/pan mượt, fit-to-window, zoom 1:1 | M | MVP |
| FR-ANN-02 | Chọn, ẩn/hiện, khóa, xóa annotation; danh sách annotation của ảnh | M | MVP |
| FR-ANN-03 | Undo/redo theo ảnh, tối thiểu 100 bước | M | MVP |
| FR-ANN-04 | Phím tắt cho tool, class, điều hướng ảnh, duyệt (theo [10](../04-giao-dien/10_desktop_ui_ux_specification.md)) | M | MVP |
| FR-ANN-05 | Autosave có debounce; chỉ báo dirty state; flush khi chuyển ảnh/đóng app | M | MVP |
| FR-ANN-06 | Hiển thị khác biệt rõ ràng giữa annotation auto (nét đứt + confidence) và human-verified | M | MVP |
| FR-BBX-01 | Vẽ, di chuyển, resize BBox; snap trong biên ảnh | M | MVP |
| FR-SEG-01 | Vẽ polygon, thêm/xóa/kéo đỉnh, đóng polygon; nhiều polygon cho một instance | M | MVP |
| FR-SEG-02 | Hiển thị mask overlay (từ RLE/polygon) có độ trong suốt điều chỉnh được | M | MVP |
| FR-SEG-03 | SAM-assisted: click điểm (+/−) hoặc vẽ box → SAM 2 trả mask → chuyển thành polygon để sửa | M | MVP |
| FR-SEG-04 | Brush/eraser chỉnh mask trực tiếp | C | V1 |
| FR-POS-01 | Đặt/kéo keypoint, đổi visibility (không gán / bị che / nhìn thấy), hiển thị skeleton | M | MVP |
| FR-POS-02 | Mỗi pose instance có BBox đi kèm (như COCO/YOLO pose) | M | MVP |

### 2.5 AI Auto-label — `FR-AI`

| Mã | Yêu cầu | Ưu tiên | Tier |
|---|---|:-:|:-:|
| FR-AI-01 | Model registry hiển thị model, task hỗ trợ, trạng thái (chưa tải / đã tải / lỗi), license, nguồn | M | MVP |
| FR-AI-02 | Tải checkpoint chỉ khi người dùng bấm tải; hiển thị nguồn, dung lượng, license; kiểm tra checksum | M | MVP |
| FR-AI-03 | Chặn tổ hợp model–task không tương thích **trước khi chạy** | M | MVP |
| FR-AI-04 | Tham số: confidence, IoU/NMS, image size, class filter, batch size; lưu được preset | M | MVP |
| FR-AI-05 | Chạy batch trên ảnh đang chọn / bộ lọc / toàn dataset; có pause/resume/cancel ở ranh giới batch | M | MVP |
| FR-AI-06 | Tiến độ theo số ảnh, throughput và ETA đo thực tế; không hiển thị % giả | M | MVP |
| FR-AI-07 | Pipeline YOLO detect → SAM 2 (prompt bằng box) → mask/polygon | M | MVP |
| FR-AI-08 | Chính sách merge kết quả vào ảnh đã có nhãn: bỏ qua / thêm mới / thay nhãn auto chưa duyệt; **không bao giờ ghi đè nhãn human-verified** | M | MVP |
| FR-AI-09 | Ghi provenance: model, checkpoint, version, tham số, job ID, thiết bị chạy | M | MVP |
| FR-AI-10 | SAM 3 concept segmentation bằng text prompt | C | V1 (điều kiện license) |
| FR-AI-11 | DINOv2 embedding: tìm ảnh tương tự, near-duplicate, gợi ý cụm | C | V1 |

### 2.6 Compute — `FR-CMP` (local) và `FR-RMT` (remote)

| Mã | Yêu cầu | Ưu tiên | Tier |
|---|---|:-:|:-:|
| FR-CMP-01 | Probe OS, CPU, RAM, ổ đĩa trống, GPU NVIDIA, VRAM, driver, CUDA runtime, PyTorch | M | MVP |
| FR-CMP-02 | Router chọn LOCAL_GPU / LOCAL_CPU / REMOTE_KAGGLE / MANUAL_REQUIRED, kèm **lý do và cảnh báo** | M | MVP |
| FR-CMP-03 | Người dùng có thể ép chế độ chạy: Auto / Force CPU / Force GPU / Force Kaggle; đặt giới hạn VRAM/RAM | M | MVP |
| FR-CMP-04 | Xử lý CUDA OOM: giảm batch có giới hạn số lần, rồi đề xuất CPU/remote; không retry vô hạn | M | MVP |
| FR-CMP-05 | Theo dõi tài nguyên (RAM/VRAM/GPU util) trong khi chạy job | S | MVP |
| FR-RMT-01 | Cấu hình Kaggle: kiểm tra CLI, kiểm tra xác thực **mà không hiển thị token** | M | MVP |
| FR-RMT-02 | Trước khi upload: hiển thị danh sách và dung lượng file sẽ gửi; người dùng phải xác nhận | M | MVP |
| FR-RMT-03 | Đóng gói input theo manifest và chunk; chỉ gửi ảnh cần thiết | M | MVP |
| FR-RMT-04 | Trạng thái job, log, timeout, retry; phát hiện session hết hạn; resume theo chunk | M | MVP |
| FR-RMT-05 | Tải artifact, kiểm tra checksum + schema + job ID trước khi import | M | MVP |
| FR-RMT-06 | Dọn dữ liệu tạm local/remote theo cài đặt | S | MVP |

### 2.7 Review & Quality — `FR-REV`, `FR-QA`

| Mã | Yêu cầu | Ưu tiên | Tier |
|---|---|:-:|:-:|
| FR-REV-01 | Review queue: chưa duyệt, đã chấp nhận, bị từ chối, đã sửa, confidence thấp, không hợp lệ | M | MVP |
| FR-REV-02 | Duyệt từng annotation hoặc cả ảnh (Accept / Reject / Edit); duyệt theo batch | M | MVP |
| FR-REV-03 | Đánh dấu "không chắc chắn" để người khác xem lại | S | MVP |
| FR-QA-01 | Validator: tọa độ ngoài ảnh, box/polygon suy biến, class ID không tồn tại, số keypoint sai, ID trùng, ảnh thiếu file | M | MVP |
| FR-QA-02 | Export preflight: chặn export nếu có lỗi mức Error; cảnh báo nếu chỉ có Warning | M | MVP |
| FR-QA-03 | Cảnh báo annotation trùng/chồng lấn cao (IoU vượt ngưỡng) | S | V1 |
| FR-QA-04 | Báo cáo phân bố class, review coverage | S | MVP |

### 2.8 Import/Export — `FR-IO`

| Mã | Yêu cầu | Ưu tiên | Tier |
|---|---|:-:|:-:|
| FR-IO-01 | Export YOLO detect / segment / pose (kèm `data.yaml`) | M | MVP |
| FR-IO-02 | Export COCO detection / instances / keypoints | M | MVP |
| FR-IO-03 | Import YOLO và COCO tương ứng; báo cáo bản ghi không ánh xạ được | M | MVP |
| FR-IO-04 | Export chỉ nhãn đã duyệt (mặc định) hoặc kèm nhãn auto (phải chọn rõ) | M | MVP |
| FR-IO-05 | Export manifest: phiên bản app, schema, thời điểm, bộ lọc, mapping class, thống kê | M | MVP |
| FR-IO-06 | Không ghi đè thư mục export đã tồn tại nếu chưa xác nhận | M | MVP |
| FR-IO-07 | Pascal VOC detection | C | V1 |

### 2.9 Vận hành — `FR-OPS`

| Mã | Yêu cầu | Ưu tiên | Tier |
|---|---|:-:|:-:|
| FR-OPS-01 | Log ứng dụng xoay vòng (rotating), có màn hình xem log, có thể xuất gói chẩn đoán đã che thông tin nhạy cảm | M | MVP |
| FR-OPS-02 | Crash recovery: phát hiện phiên bị ngắt, khôi phục job đang chạy về trạng thái INTERRUPTED | M | MVP |
| FR-OPS-03 | Dọn cache (thumbnail, model, temp) có hiển thị dung lượng | S | MVP |
| FR-OPS-04 | Bản portable Windows | M | MVP |
| FR-OPS-05 | Installer Windows, cơ chế kiểm tra cập nhật | S | V1 |

## 3. Quy tắc nghiệp vụ (BR)

| Mã | Quy tắc | Xử lý khi vi phạm |
|---|---|---|
| BR-01 | Annotation do model tạo luôn có `origin=auto` và `review_status=unreviewed` cho đến khi người duyệt | Không được export như nhãn đã duyệt |
| BR-02 | Auto-label **không bao giờ** sửa hoặc xóa annotation có `review_status ∈ {accepted, corrected}` hoặc `origin ∈ {manual, imported}` | Bỏ qua kết quả trùng, ghi vào log job |
| BR-03 | Sửa một annotation auto sẽ chuyển `origin=corrected` và giữ liên kết tới bản dự đoán gốc | — |
| BR-04 | Không truyền dữ liệu ra ngoài nếu chưa có xác nhận cho **đúng job đó** (consent không dùng lại cho job khác) | Job dừng ở NEEDS_ATTENTION |
| BR-05 | Model chỉ được chạy với task mà adapter khai báo hỗ trợ | Chặn trước khi tạo job |
| BR-06 | Ảnh nguồn là read-only: app không bao giờ sửa, đổi tên hay xóa ảnh nguồn | — |
| BR-07 | Class ID nội bộ bất biến; chỉ số class khi export được tính lúc export, theo mapping | — |
| BR-08 | Mọi tọa độ canonical là pixel, theo ảnh **đã áp dụng EXIF orientation** (xem [06](../03-du-lieu/06_annotation_data_model.md)) | Validator báo lỗi |
| BR-09 | Kết quả remote chỉ được import khi khớp checksum, schema và job ID | Job → NEEDS_ATTENTION, không import |
| BR-10 | Một job chỉ ghi kết quả vào ảnh nằm trong phạm vi (scope) đã được chốt khi tạo job | Bỏ kết quả lạ, ghi cảnh báo |
| BR-11 | Xóa class đang được dùng phải chọn hành động rõ ràng (FR-CLS-02) | Chặn thao tác |
| BR-12 | Export mặc định chỉ gồm annotation `accepted`/`corrected`/`manual`/`imported` | — |

## 4. Yêu cầu phi chức năng (NFR)

| Mã | Nhóm | Yêu cầu | Cách kiểm chứng |
|---|---|---|---|
| NFR-PERF-01 | Hiệu năng | Không có thao tác nào chặn UI thread quá 100 ms; inference và I/O nặng chạy ngoài UI thread | Watchdog event-loop trong test |
| NFR-PERF-02 | Hiệu năng | Chuyển ảnh trong workspace: mục tiêu ≤ 300 ms với ảnh ≤ 12 MP trên SSD (PROPOSAL, cần benchmark) | Benchmark |
| NFR-PERF-03 | Hiệu năng | Canvas vẫn tương tác được với ≥ 500 annotation/ảnh (PROPOSAL) | Benchmark UI |
| NFR-PERF-04 | Hiệu năng | Trình duyệt dataset không nạp toàn bộ ảnh hay annotation vào RAM; truy vấn có phân trang | Review code + test 50k ảnh tổng hợp |
| NFR-REL-01 | Tin cậy | Ghi annotation theo transaction; DB ở chế độ WAL; mất tối đa phần chưa flush trong khoảng debounce | Kill test |
| NFR-REL-02 | Tin cậy | Đóng app không bỏ lại tiến trình worker mồ côi | Test shutdown |
| NFR-REL-03 | Tin cậy | Migration có backup trước và xử lý được khi bị ngắt giữa chừng | Migration test |
| NFR-SEC-01 | Bảo mật | Không lưu token trong source, log, config của project; log được redact | Secret scan + test redact |
| NFR-SEC-02 | Bảo mật | Gọi subprocess bằng argument list, không qua shell; validate đường dẫn | Review + test |
| NFR-PRIV-01 | Riêng tư | Mặc định không có kết nối mạng nào ngoài: tải model (do người dùng bấm) và remote job (đã xác nhận) | Test mạng |
| NFR-USA-01 | Khả dụng | Mọi màn hình có trạng thái loading / empty / error / success; thông báo lỗi có hướng xử lý | Review UI |
| NFR-USA-02 | Khả dụng | Toàn bộ luồng chính thao tác được bằng bàn phím | Test thủ công theo checklist |
| NFR-PORT-01 | Di động | Project dùng đường dẫn tương đối với dataset root; chuyển máy thì relink được | Test relink |
| NFR-MAINT-01 | Bảo trì | Thêm model mới chỉ cần viết adapter + đăng ký, không sửa core/UI | Contract test adapter |
| NFR-MAINT-02 | Bảo trì | Core không import PySide6; adapter không import UI | Kiểm tra import boundary tự động |
| NFR-OBS-01 | Quan sát | Mỗi job có job ID và correlation ID xuyên suốt log local/remote | Test log |

## 5. Tiêu chí nghiệm thu chính (Given/When/Then)

| Mã | Tiêu chí |
|---|---|
| AC-01 | **Given** một thư mục 1.000 ảnh lồng 3 cấp, **When** import, **Then** có 1.000 bản ghi (trừ file lỗi được liệt kê), UI vẫn tương tác được trong lúc import, và import lại không tạo bản ghi trùng. |
| AC-02 | **Given** model YOLO segment đã tải, **When** chọn task POSE, **Then** nút chạy bị vô hiệu hóa kèm lý do "Model không hỗ trợ task POSE". |
| AC-03 | **Given** một ảnh có 2 box đã được duyệt, **When** chạy auto-label cho ảnh đó, **Then** 2 box đó giữ nguyên, kết quả mới chỉ được thêm dưới dạng `unreviewed`. |
| AC-04 | **Given** máy không có GPU và chưa cấu hình Kaggle, **When** chạy SAM 2 cho 5.000 ảnh, **Then** router trả về LOCAL_CPU kèm cảnh báo tốc độ, hoặc MANUAL_REQUIRED kèm hướng dẫn cấu hình remote; không upload gì. |
| AC-05 | **Given** job Kaggle bị hết phiên giữa chừng, **When** người dùng bấm Resume, **Then** chỉ các chunk chưa hoàn thành được gửi lại và không có annotation bị trùng. |
| AC-06 | **Given** dataset COCO keypoints hợp lệ, **When** import → export COCO, **Then** keypoints, visibility, bbox, category và skeleton tương đương với bản gốc (so sánh có dung sai làm tròn ≤ 1e-3 px). |
| AC-07 | **Given** đang chỉnh polygon, **When** kill process, **Then** mở lại app thấy mọi thay đổi trước thời điểm kill hơn 2 giây. |
| AC-08 | **Given** có annotation với box vượt ngoài ảnh, **When** export, **Then** preflight báo lỗi kèm ảnh/annotation cụ thể và chặn export cho tới khi được sửa hoặc người dùng chọn clip tự động có ghi nhận. |

## 6. Truy vết

Ma trận Requirement → Issue → Test sẽ được tạo ở Bước 04 (`docs/06-ke-hoach/project-management/REQUIREMENTS_TRACEABILITY.md`).
