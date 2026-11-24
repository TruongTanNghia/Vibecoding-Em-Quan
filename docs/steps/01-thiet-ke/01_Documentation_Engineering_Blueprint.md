# PROMPT 01 — DOCUMENTATION & ENGINEERING BLUEPRINT
## Phân tích, nghiên cứu, thiết kế kiến trúc và lập kế hoạch toàn bộ sản phẩm

> **Bước 01/07 · Giai đoạn 1 — Thiết kế & Tài liệu** · Đầu vào: ý tưởng sản phẩm · Đầu ra: `README.md`, `docs/01-tong-quan/` … `docs/07-van-hanh/` (tài liệu 00–19, mục lục: `docs/README.md`)
> Bước tiếp theo: [02 — System Architecture & Diagrams](02_System_Architecture_Diagrams.md)

```text
# MASTER PROMPT 01: DESKTOP AI AUTO ANNOTATION — DOCUMENTATION & ENGINEERING BLUEPRINT

## 1. VAI TRÒ

Bạn là Principal Software Architect, Senior Desktop Application Engineer, Senior Computer Vision Engineer, AI/ML Engineer, MLOps Engineer, UX Architect và Technical Lead.

Nhiệm vụ của bạn là phân tích, nghiên cứu tính khả thi, thiết kế kiến trúc và tạo bộ tài liệu kỹ thuật hoàn chỉnh cho một ứng dụng desktop chuyên nghiệp dùng để quản lý dataset và tự động/hỗ trợ gán nhãn dữ liệu Computer Vision.

Bạn phải tư duy như người chịu trách nhiệm đưa sản phẩm từ ý tưởng đến một ứng dụng desktop có thể cài đặt, sử dụng thực tế, mở rộng, kiểm thử và bảo trì.

Không bắt đầu viết toàn bộ source code trong giai đoạn này. Đây là giai đoạn DOCUMENTATION FIRST.

## 2. THÔNG TIN DỰ ÁN

Tên dự án: AutoLabel Studio (tên đề xuất, có thể điều chỉnh).

Loại sản phẩm:
- Ứng dụng desktop chạy trên máy tính cá nhân.
- Ưu tiên xử lý dữ liệu và inference trực tiếp trên máy (Local-first).
- Có cơ chế chuyển tác vụ inference sang máy chủ GPU từ xa thông qua CLI khi phần cứng local không đáp ứng.
- Không yêu cầu người dùng mở giao diện Kaggle Notebook hoặc thao tác thủ công trên trình duyệt để chạy inference từ xa.

Mục tiêu:
Xây dựng công cụ hỗ trợ toàn bộ vòng đời annotation:
Import dữ liệu → Quản lý dataset → Chọn loại annotation → Chọn model → Auto-label → Hậu xử lý → Human review → Chỉnh sửa → Validation → Export dataset.

Các loại annotation bắt buộc:
1. Object Detection — Bounding Box (BBox).
2. Instance Segmentation / Polygon / Mask (SEG).
3. Human Pose / Keypoints (POSE).

Các loại mở rộng có thể xem xét:
- Image Classification.
- Semantic Segmentation.
- Oriented Bounding Box (OBB).
- Video annotation và tracking.
- Keypoint schema tùy biến.
Chỉ đưa vào MVP nếu tính khả thi và phạm vi cho phép.

Các model/công nghệ người dùng ưu tiên nghiên cứu:
- Ultralytics YOLO: detection, segmentation, pose; chọn đúng checkpoint/task tương ứng.
- Meta Segment Anything Model 2 (SAM 2).
- Segment Anything Model 3 (SAM 3), nếu quyền truy cập, package, checkpoint và license phù hợp.
- DINOv2.
- DINOv3, nếu có implementation/checkpoint chính thức phù hợp và điều kiện sử dụng được xác minh.

Nguyên tắc sử dụng model:
- Ưu tiên pretrained model/checkpoint có sẵn trước khi nghĩ đến huấn luyện model mới.
- Không coi mọi model là có thể thay thế trực tiếp cho nhau.
- YOLO detection/segmentation/pose là các task/model head khác nhau.
- SAM là mô hình segmentation/promptable segmentation; không mặc định SAM tự sinh class name hoặc thay thế detector.
- DINOv2/DINOv3 chủ yếu là visual representation/feature backbone. Chỉ dùng cho embedding, similarity, retrieval, clustering, hỗ trợ phân nhóm hoặc workflow phù hợp; không mô tả chúng như detector/pose estimator nếu không có head/pipeline tương ứng.
- Có thể kết hợp detector → SAM refinement → annotation review.
- Thiết kế model adapter/plugin để bổ sung model mới mà không phải sửa lõi ứng dụng.

Phần cứng:
- Ưu tiên CPU/GPU/RAM/VRAM/ổ cứng đang có trên máy người dùng.
- Ứng dụng phải kiểm tra phần cứng, CUDA, driver và VRAM trước khi chạy model.
- Có chế độ CPU fallback cho tác vụ phù hợp, nhưng phải cảnh báo về tốc độ.
- Nếu local không có GPU hoặc không đủ tài nguyên, có thể chuyển inference sang Kaggle GPU thông qua CLI.
- Kaggle chỉ là compute worker từ xa, không phải giao diện chính của sản phẩm.
- Desktop app vẫn là nơi quản lý project, dataset, annotation, review và export.
- Remote execution phải có cấu hình, trạng thái job, log, retry, timeout, đồng bộ artifact và xử lý session bị ngắt.
- Không giả định Kaggle GPU luôn có sẵn, luôn miễn phí, không giới hạn hoặc có quota cố định. Phải kiểm tra chính sách hiện hành và ghi rõ giới hạn cần người dùng xác nhận.
- Nếu Kaggle không khả dụng, thiết kế provider interface để sau này có thể tích hợp máy chủ GPU/worker khác; không triển khai cloud provider ngoài phạm vi nếu chưa được duyệt.

Hệ điều hành mục tiêu:
Ưu tiên Windows desktop; đánh giá khả năng hỗ trợ Linux. macOS chỉ đưa vào roadmap nếu phù hợp.

Ngôn ngữ ưu tiên: Python.

Framework desktop:
Đánh giá PySide6/Qt hoặc lựa chọn tương đương. Không chốt framework trước khi phân tích UI phức tạp, hiệu năng hiển thị ảnh, khả năng đóng gói, license và trải nghiệm phát triển.

Các thông tin chưa có:
- Cấu hình máy mục tiêu.
- Quy mô dataset điển hình.
- Định dạng input/output cần ưu tiên.
- Có cần video annotation ở MVP hay không.
- Có cần lưu project nhiều người dùng hay chỉ single-user local.
- Tên thương hiệu cuối cùng.
Hãy không hỏi lại điều đã có. Chỉ hỏi tối đa 5 câu có ảnh hưởng lớn đến kiến trúc; nếu có thể tiếp tục, ghi giả định rõ ràng.

## 3. NGUYÊN TẮC THIẾT KẾ SẢN PHẨM

### 3.1 Desktop-first và Local-first

- UI desktop là trung tâm điều khiển.
- Dataset mặc định nằm trên ổ đĩa local do người dùng chọn.
- Không upload dữ liệu lên dịch vụ bên ngoài nếu người dùng chưa chủ động chọn remote inference.
- Có thể chọn thư mục dữ liệu, project workspace, model cache và thư mục export.
- Không được mặc định copy toàn bộ dataset vào database.
- Database chỉ lưu metadata, project config, annotation metadata, task state và đường dẫn/URI nội bộ cần thiết.
- Có chiến lược xử lý file di chuyển, đổi tên, thiếu file và ổ đĩa ngoài bị ngắt.
- Không khóa người dùng vào một định dạng annotation độc quyền.

### 3.2 Annotation workflow

Thiết kế workflow có trạng thái rõ ràng:
- Imported
- Ready
- Queued
- Running
- Auto-labeled
- Needs Review
- Reviewed
- Corrected
- Validated
- Exported
- Failed

Cho phép:
- Chọn một ảnh hoặc batch ảnh.
- Chọn annotation task: BBox, SEG hoặc POSE.
- Chọn model và checkpoint phù hợp.
- Cấu hình confidence threshold, IoU/NMS hoặc tham số riêng của model.
- Chạy auto-label theo batch.
- Pause/resume/cancel job khi khả thi.
- Hiển thị progress, throughput, ETA nếu có cơ sở đo.
- Xem kết quả overlay trên ảnh.
- Sửa, thêm, xóa, đổi class và điều chỉnh tọa độ.
- Duyệt từng ảnh hoặc theo batch.
- Lọc ảnh theo trạng thái, class, lỗi và confidence.
- Lưu tự động, undo/redo.
- Đánh dấu nhãn chưa chắc chắn để human review.

Không được để kết quả model tự động được xem là ground truth. Phải phân biệt Auto-generated annotation và Human-verified annotation.

### 3.3 Annotation data model

Thiết kế schema trung lập với định dạng export, có khả năng biểu diễn:
- Image asset, dimensions, checksum, relative path.
- Dataset, project, class/category.
- Bounding box (quy ước tọa độ rõ ràng).
- Polygon và segmentation mask.
- Keypoints, skeleton edges, visibility flags và keypoint schema.
- Annotation ID, source model, model version, confidence.
- Annotation origin: manual / auto / imported / corrected.
- Review status, reviewer metadata nếu có.
- History/version hoặc audit event phù hợp.

Quy định rõ:
- Pixel coordinates hay normalized coordinates.
- XYXY hay XYWH.
- Quy ước thứ tự keypoints.
- Cách biểu diễn polygon, holes, mask và visibility.
- Cách xử lý ảnh EXIF orientation.
- Cách xử lý ảnh grayscale, alpha channel và ảnh quá lớn.
- Cách tránh mất thông tin khi import/export.

### 3.4 Model adapter architecture

Tách model integration khỏi UI và annotation core.

Mỗi adapter cần khai báo:
- Model ID, display name, version.
- Supported tasks: detect / segment / pose / embedding / prompt segmentation.
- Input requirements.
- Output schema.
- Checkpoint/config requirements.
- Device support: CPU/CUDA/khác.
- VRAM/RAM ước lượng nếu có dữ liệu đáng tin.
- Model license và source.
- Download/cache strategy.
- Inference parameters.
- Warmup, batch support, cancellation capability.
- Error mapping.

Thiết kế adapter cho:
- YOLO detection.
- YOLO segmentation.
- YOLO pose.
- SAM 2.
- SAM 3 nếu khả thi và được xác minh.
- DINOv2/DINOv3 feature extraction hoặc similarity workflows.

Đánh giá các pipeline kết hợp:
A. YOLO detect → BBox annotation.
B. YOLO detect → SAM prompt/refinement → mask/SEG annotation.
C. YOLO pose → keypoint annotation.
D. SAM prompt-assisted segmentation → user review.
E. DINO embeddings → similarity search/duplicate discovery/cluster suggestions, nếu phù hợp.

Không ép mọi model vào một interface thiếu chính xác. Có thể có task-specific interfaces cùng một registry chung.

## 4. LOCAL HARDWARE DETECTION & COMPUTE ROUTER

Đây là yêu cầu kiến trúc trọng yếu.

Thiết kế Hardware & Compute Manager để:
1. Detect OS, CPU, RAM, free disk.
2. Detect NVIDIA GPU, VRAM, driver, CUDA runtime khi có thể.
3. Detect các accelerator khác nếu framework hỗ trợ thực tế.
4. Kiểm tra Python/PyTorch/model dependencies.
5. Chạy capability probe/smoke test an toàn.
6. Đánh giá model-task có thể chạy trên CPU/GPU local hay không.
7. Chọn device theo policy và cấu hình người dùng.
8. Kiểm tra free VRAM trước khi load model và trong lúc xử lý nếu có thể.
9. Có giới hạn batch size, image size, concurrency.
10. Xử lý CUDA OOM: giảm batch, giải phóng cache có kiểm soát, retry theo policy; không retry vô hạn.
11. Ghi rõ lý do chọn local hoặc remote.

Định nghĩa các execution modes:
- LOCAL_GPU: GPU local đủ điều kiện.
- LOCAL_CPU: chỉ dùng CPU cho tác vụ phù hợp.
- REMOTE_KAGGLE: gửi inference job qua Kaggle CLI.
- MANUAL_REQUIRED: không có execution path khả dụng hoặc cần người dùng quyết định.

Không dùng một ngưỡng VRAM tùy tiện cho tất cả model. Xây dựng capability profile theo model, task, image size, precision, batch size; ban đầu có thể dùng ước lượng bảo thủ và benchmark thực nghiệm.

Cho phép người dùng:
- Auto select.
- Force local CPU.
- Force local GPU.
- Force remote Kaggle.
- Chỉ định giới hạn VRAM/RAM.
- Chọn remote profile.
- Xem lý do và cảnh báo trước khi chạy.

## 5. REMOTE KAGGLE CLI WORKER

Kaggle được sử dụng như compute backend từ xa, không phải nơi chứa logic UI.

Thiết kế workflow dự kiến:
Desktop App
→ Job Planner
→ Package input manifest/config
→ Prepare dataset subset hoặc archive
→ Upload/sync input theo phương thức Kaggle CLI/API được xác minh
→ Create/execute Kaggle notebook/script/job
→ Load model/checkpoint
→ Run inference
→ Write predictions + logs + metadata
→ Package output artifacts
→ Download/sync result
→ Validate output
→ Import annotations vào desktop project
→ Human review.

Yêu cầu:
- Chỉ gửi các file cần thiết, không mặc định gửi toàn bộ dataset.
- Hỗ trợ batch/chunking và manifest để xử lý dataset lớn.
- Có resume/retry và nhận diện job đã chạy để tránh xử lý trùng.
- Có job ID, correlation ID, trạng thái và log.
- Có timeout, cancel best-effort và phát hiện session expired.
- Kiểm tra artifact tồn tại, checksum và schema trước khi import.
- Không đồng bộ annotation chưa hoàn chỉnh như kết quả hợp lệ.
- Có thể chọn export kết quả từng batch.
- Giải thích rõ Kaggle session có thể bị ngắt, quota/availability thay đổi và dữ liệu tạm có thể bị xóa.
- Kiểm tra điều khoản, giới hạn sử dụng, quyền dữ liệu và license của model.
- Không lưu Kaggle token trong source code, logs, issue hoặc file cấu hình công khai.
- Dùng secret store hoặc environment variable; hướng dẫn cấu hình an toàn.
- Không yêu cầu người dùng cung cấp token trong chat.
- Không giả định Kaggle CLI hỗ trợ một thao tác cụ thể nếu chưa xác minh tài liệu hiện hành.
- Tách Kaggle adapter khỏi core để có thể thay thế bằng SSH worker hoặc provider khác sau này.

## 6. PHẠM VI CHỨC NĂNG CẦN NGHIÊN CỨU

Phân nhóm theo MVP, V1 và Future:

A. Project & Dataset
- Create/open/close project.
- Import folder/images.
- Scan nested folders.
- File preview, thumbnail cache.
- Duplicate detection bằng hash tùy chọn.
- Dataset statistics.
- Class management.
- Dataset integrity checks.

B. Annotation Workspace
- Canvas hiển thị ảnh có zoom/pan.
- BBox draw/edit/move/resize.
- Polygon draw/edit/vertex operations.
- Mask overlay/brush tools nếu khả thi.
- Pose keypoints và skeleton editor.
- Class selector.
- Selection, visibility, lock, delete.
- Undo/redo.
- Keyboard shortcuts.
- Autosave và dirty-state indicator.

C. AI Auto Label
- Model registry.
- Model download/cache manager.
- YOLO detection/segmentation/pose.
- SAM 2/SAM 3 integration phù hợp.
- DINO embedding workflows phù hợp.
- Batch inference.
- Model parameter presets.
- Per-model logs and errors.
- Confidence filtering.
- Human-in-the-loop review.

D. Compute Management
- Hardware detection.
- Local device selection.
- Resource monitor.
- Inference queue.
- Remote Kaggle CLI worker.
- Job state synchronization.
- Retry/resume.
- Artifact transfer.

E. Quality Assurance
- Missing annotation check.
- Invalid coordinate check.
- Degenerate bbox/polygon check.
- Out-of-bounds geometry.
- Invalid class IDs.
- Invalid keypoint count/order.
- Duplicate/overlapping annotations warning.
- Empty label/image mismatch.
- Class distribution.
- Review coverage.
- Export preflight.

F. Import/Export
- YOLO detection.
- YOLO segmentation.
- YOLO pose.
- COCO detection/instances/keypoints where supported.
- Pascal VOC where applicable.
- Export manifest and class mapping.
- Round-trip import/export tests.
- Không hứa hỗ trợ format nếu chưa xác định schema và test.

G. Packaging & Operations
- Windows installer/portable build.
- Model download management.
- Cache cleanup.
- App logs.
- Crash recovery.
- Backup/restore project metadata.
- Version migration.
- Update strategy.

## 7. NGHIÊN CỨU VÀ XÁC MINH CÔNG NGHỆ

Khi có Internet:
- Ưu tiên official docs, official GitHub repositories, papers gốc và model cards.
- Xác minh trạng thái, phiên bản, checkpoint, license, yêu cầu phần cứng và cách cài đặt của YOLO, SAM 2, SAM 3, DINOv2, DINOv3.
- Phân biệt model architecture, checkpoint, library wrapper và license của từng thành phần.
- Xác minh khả năng thương mại/phân phối checkpoint.
- Xác minh PySide6/Qt, PyTorch, Ultralytics, SAM dependencies, ONNX Runtime nếu được đề xuất.
- Xác minh Kaggle CLI/API workflow và chính sách sử dụng GPU hiện hành.
- Không suy đoán DINOv3 có package/checkpoint/weights giống DINOv2.
- Nếu không thể truy cập nguồn, ghi UNVERIFIED và nêu bước kiểm chứng.

Đánh giá desktop stack:
- PySide6/Qt.
- PyQt nếu có lý do.
- Electron/Tauri chỉ so sánh khi có lợi ích rõ ràng; không chọn chỉ vì phổ biến.
- Worker thread/process để inference không block UI.
- SQLite hoặc lựa chọn metadata store phù hợp.
- OpenCV/Pillow/imageio cho image IO nếu cần.
- QGraphicsView hoặc canvas strategy phù hợp với lượng annotation lớn.
- Packaging bằng PyInstaller/Nuitka hoặc giải pháp phù hợp sau khi kiểm chứng.

## 8. CẤU TRÚC TÀI LIỆU BẮT BUỘC

Tạo `docs/` gồm:

00. `00_project_overview.md`
- Vision, problem, target users, scope, non-goals, MVP, success criteria.

01. `01_requirements.md`
- User stories, functional/non-functional requirements, BBox/SEG/POSE, import/export, desktop UX, local/remote compute, acceptance criteria.

02. `02_feasibility_study.md`
- Technical, hardware, dataset scale, model, packaging, Kaggle remote, cost, risks, MVP feasibility.

03. `03_research_and_references.md`
- YOLO, SAM 2, SAM 3, DINOv2, DINOv3, desktop frameworks, annotation formats, Kaggle CLI, licenses and official sources.

04. `04_system_architecture.md`
- Component diagram, data flow, annotation workflow, local inference, remote Kaggle CLI flow, job state, persistence, error handling.

05. `05_model_and_inference_design.md`
- Model registry, task/model compatibility matrix, adapters, pipeline compositions, model lifecycle, caching, inference parameters, fallback.

06. `06_annotation_data_model.md`
- Canonical annotation schema, geometry, coordinate conventions, keypoints, masks, provenance, review state, serialization.

07. `07_dataset_and_import_export.md`
- Dataset structure, import, image metadata, supported formats, validation, class mapping, export and round-trip.

08. `08_hardware_and_compute_routing.md`
- Hardware probe, resource profiles, device policy, CPU/GPU selection, OOM strategy, remote trigger criteria.

09. `09_kaggle_remote_worker.md`
- CLI setup, credentials, packaging, transfer, remote execution, logs, retry/resume, artifact validation, security and quota limitations.

10. `10_desktop_ui_ux_specification.md`
- Screen map, navigation, annotation workspace, keyboard shortcuts, states, error messages, accessibility, wireframe descriptions.

11. `11_environment_and_setup.md`
- OS, Python, dependency matrix, CUDA compatibility, setup, smoke test, local and Kaggle CLI setup.

12. `12_codebase_structure.md`
- Folder tree, module responsibilities, import boundaries, entry points, config and data locations.

13. `13_implementation_roadmap.md`
- Milestones, tasks, dependencies, DoD, risks, effort assumptions.

14. `14_testing_and_quality_strategy.md`
- Unit, integration, UI, model adapter, geometry, import/export, local GPU, remote worker, performance and recovery tests.

15. `15_packaging_release_and_operations.md`
- Windows packaging, model assets, cache, updates, logs, crash recovery, backup and release.

16. `16_security_privacy_and_licenses.md`
- Local data privacy, Kaggle credentials, data transfer consent, model/library licenses, secrets and threat model.

17. `17_troubleshooting.md`
- Dependency, CUDA, VRAM, image IO, annotation corruption, Kaggle session, transfer, packaging.

18. `18_architecture_decision_records.md`
- ADR records with context, options, decision, trade-offs and status.

19. `19_project_readiness_checklist.md`
- NOT STARTED / IN PROGRESS / BLOCKED / DONE / N/A.

README.md:
- Summary, features, architecture, supported tasks/models, setup, run, screenshots placeholders only if absent, formats, limitations, docs links.

## 9. KIẾN TRÚC CẦN THỂ HIỆN

Tối thiểu phải đánh giá và mô tả các thành phần:
- Desktop UI Layer.
- Application/Use Case Layer.
- Project & Dataset Service.
- Annotation Core / Geometry Engine.
- Annotation Persistence.
- Model Registry & Model Adapters.
- Inference Orchestrator.
- Hardware Detector & Compute Router.
- Local Inference Worker.
- Remote Kaggle CLI Worker.
- Job Queue & State Store.
- Import/Export Adapters.
- Validation & Quality Engine.
- Logging/Diagnostics.

Sơ đồ Mermaid bắt buộc:
1. Overall component architecture.
2. User annotation workflow.
3. Local inference sequence.
4. Remote Kaggle CLI sequence.
5. Compute routing decision flow.
6. Annotation state machine.

UI thread không được chạy inference nặng trực tiếp. Đánh giá QThread, multiprocessing hoặc worker process phù hợp; quản lý model lifecycle và GPU context an toàn.

## 10. ROADMAP ĐỊNH HƯỚNG

Đề xuất milestone phù hợp, có thể điều chỉnh:

M0 — Requirements, research, architecture.
M1 — Desktop shell, project workspace, settings, logging.
M2 — Dataset import, image browser, metadata store.
M3 — Annotation core: BBox.
M4 — SEG and POSE editors + canonical schema.
M5 — Model registry and local YOLO inference.
M6 — SAM integration and model composition.
M7 — Hardware detection and compute routing.
M8 — Kaggle CLI remote worker.
M9 — Review, validation, import/export.
M10 — Performance, recovery, packaging, release.

Không bắt buộc giữ nguyên số milestone. Phải chỉ ra dependency và đường đi MVP ngắn nhất.

## 11. TIÊU CHUẨN CHẤT LƯỢNG

- Không bịa kết quả benchmark, model compatibility, version, license hoặc Kaggle quota.
- Phân biệt CONFIRMED / ASSUMPTION / PROPOSAL / UNVERIFIED.
- Mỗi lựa chọn kỹ thuật phải có lý do, trade-off và phương án thay thế.
- Không over-engineering.
- Không buộc phải huấn luyện model mới; ưu tiên pretrained inference.
- Không để inference làm treo UI.
- Không mất annotation khi app crash hoặc đóng bất thường.
- Không tự động ghi đè nhãn human-verified bằng kết quả auto-label.
- Không gửi dữ liệu ra ngoài mà không có hành động xác nhận rõ ràng.
- Không đặt Kaggle token trong source, logs hoặc README.
- Không giả định model có thể dùng chung một output schema mà không có adapter.
- Phải có test round-trip import/export và test geometry.
- Phải định nghĩa cách đo hiệu năng: thời gian/ảnh, throughput, RAM, VRAM, UI responsiveness và transfer time.

## 12. QUY TRÌNH THỰC HIỆN

1. Phân tích yêu cầu và unknowns.
2. Đề xuất MVP và non-goals.
3. Nghiên cứu công nghệ/model/license.
4. Thiết kế annotation schema.
5. Thiết kế desktop architecture và UX.
6. Thiết kế model adapters.
7. Thiết kế local hardware detection và compute router.
8. Thiết kế remote Kaggle CLI worker.
9. Thiết kế testing, packaging và roadmap.
10. Sinh toàn bộ tài liệu Markdown.
11. Kiểm tra chéo tài liệu và dependency.
12. Tổng hợp quyết định cần chủ dự án phê duyệt.

Không triển khai toàn bộ code trong prompt này. Nếu có thể tạo file, hãy tạo trực tiếp đúng đường dẫn và báo cáo file thực tế đã tạo.

## 13. BÁO CÁO CUỐI

Báo cáo:
- Project scope và MVP.
- Kiến trúc đề xuất.
- Model/task compatibility matrix.
- Annotation schema.
- Local-vs-remote routing policy.
- Kaggle CLI workflow.
- Công nghệ đã xác minh và chưa xác minh.
- Danh sách docs được tạo.
- Rủi ro và câu hỏi cần chốt.
- Issue/roadmap đầu tiên nên thực hiện.

Bắt đầu bằng việc khảo sát repository hiện tại. Nếu chưa có source code, làm việc trên trạng thái greenfield và không giả vờ đã có tính năng.
```

---
