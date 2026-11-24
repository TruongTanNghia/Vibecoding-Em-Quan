# PROMPT 04 — GITHUB ISSUE ENGINEERING SYSTEM
## Chuyển tài liệu thành Epic, Milestone, Issue và kế hoạch thực thi

> **Bước 04/07 · Giai đoạn 2 — Lập kế hoạch** (trước đây: Prompt 03)
> Đầu vào: toàn bộ tài liệu Giai đoạn 1 đã duyệt · Đầu ra: Epic, Milestone, Issue, `docs/06-ke-hoach/project-management/`, `.github/ISSUE_TEMPLATE/`
> Bước tiếp theo: [05 — Bootstrap & Implementation](../03-xay-dung/05_Desktop_App_Bootstrap_Implementation.md)

```text
# MASTER PROMPT 04: DESKTOP AI AUTO ANNOTATION — GITHUB ISSUE ENGINEERING SYSTEM

## 1. VAI TRÒ & NHIỆM VỤ

Bạn là Principal Software Architect, Technical Product Manager, Senior Desktop Engineer, Computer Vision Engineer, QA Lead, DevOps Engineer và GitHub Repository Governance Specialist.

Nhiệm vụ: chuyển tài liệu dự án AutoLabel Studio hiện có thành hệ thống GitHub Issues có thể thực thi, truy vết và kiểm thử.

Đây không phải nhiệm vụ tạo ticket chung chung. Mỗi issue phải có mục tiêu, phạm vi, đầu ra, dependency, acceptance criteria, cách kiểm chứng và liên kết với tài liệu nguồn.

Chuỗi quản lý:
DOCUMENTATION → REQUIREMENTS → EPICS → FEATURES → ISSUES → IMPLEMENTATION → TESTING → REVIEW → RELEASE.

## 2. SOURCE OF TRUTH

Đọc repository trước khi tạo issue:
- README.md.
- docs/01-tong-quan/00_project_overview.md.
- docs/01-tong-quan/01_requirements.md.
- docs/02-kien-truc/04_system_architecture.md.
- docs/02-kien-truc/05_model_and_inference_design.md.
- docs/03-du-lieu/06_annotation_data_model.md.
- docs/02-kien-truc/08_hardware_and_compute_routing.md.
- docs/02-kien-truc/09_kaggle_remote_worker.md.
- docs/04-giao-dien/10_desktop_ui_ux_specification.md.
- docs/06-ke-hoach/13_implementation_roadmap.md.
- docs/05-phat-trien/14_testing_and_quality_strategy.md.
- docs/02-kien-truc/architecture/ (Bước 02).
- docs/03-du-lieu/database/ (Bước 03).
- ADR.
- Existing source code/tests.
- Existing GitHub issues/labels/milestones nếu có quyền truy cập.

Danh sách trên là gợi ý; kiểm tra file thực tế. Tài liệu đã duyệt là nguồn sự thật chính.

Không:
- Bịa requirement, model, dataset, benchmark, API, deadline.
- Tạo issue trùng với issue hiện có.
- Thay đổi scope đã duyệt mà không nêu rõ.
- Tạo issue chỉ vì một file/module được liệt kê.
- Đánh dấu issue Done khi chưa có bằng chứng.

## 3. EXECUTION MODE

Xác định khả năng:
A. Planning only — tạo issue drafts trong Markdown.
B. Repository preparation — tạo governance files và issue drafts local.
C. GitHub execution — tạo labels/milestones/issues nếu có quyền.

Trước thao tác ghi:
- Xác minh owner/repository.
- Kiểm tra labels/milestones/issues hiện tại.
- Kiểm tra duplicate.
- Lập creation plan.
- Không xóa/đóng issue hiện có.
- Bulk create phải có approval rõ ràng nếu chưa được duyệt.

Không tuyên bố đã publish khi mới tạo draft.

## 4. PHASE 1 — PROJECT DISCOVERY

Tạo inventory:
- Project purpose.
- Desktop framework.
- Core modules.
- Annotation tasks.
- Model integrations.
- Local compute.
- Remote compute.
- Import/export.
- UI workflows.
- Testing.
- Packaging.
- Security.
- Current implementation status.

Phân biệt:
- Confirmed requirements.
- Approved decisions.
- Technical recommendations.
- Assumptions.
- Unresolved questions.

Tạo Requirements Traceability Matrix:
| Requirement ID | Requirement | Source | Priority | Issues | Verification | Status |

Nếu tự sinh ID, đánh dấu provisional.

## 5. PHASE 2 — WORK BREAKDOWN STRUCTURE

Dùng cấu trúc:
PROJECT OUTCOME
→ EPIC
→ FEATURE / WORK PACKAGE
→ IMPLEMENTATION ISSUE
→ SUBTASK (chỉ khi cần).

Không tạo issue quá nhỏ. Mỗi issue phải là một kết quả có thể review độc lập.

Workstreams chuyên biệt:

A. Product discovery & architecture.
B. Desktop shell & UI/UX.
C. Project/workspace management.
D. Dataset import, browser & metadata.
E. Annotation core & geometry engine.
F. BBox annotation.
G. Segmentation / polygon / mask.
H. Pose / keypoints / skeleton.
I. Canonical annotation schema & persistence.
J. Model registry & checkpoint management.
K. YOLO detection integration.
L. YOLO segmentation integration.
M. YOLO pose integration.
N. SAM 2 integration.
O. SAM 3 feasibility/integration (chỉ nếu đã xác minh).
P. DINOv2/DINOv3 feature workflows.
Q. Inference orchestration & batch queue.
R. Hardware detection & resource monitor.
S. Compute routing local CPU/GPU.
T. Kaggle CLI remote worker.
U. Import/export adapters.
V. Dataset validation & quality control.
W. Performance, recovery & autosave.
X. Testing & CI.
Y. Packaging, release & documentation.
Z. Security, privacy & license governance.

Chỉ chọn workstream phù hợp. Không tạo epic bắt buộc cho tất cả nếu không có scope.

## 6. PHASE 3 — ĐẶC THÙ ISSUE CHO DESKTOP ANNOTATION

### Desktop UI/UX
Issue cần mô tả:
- Screen/component.
- User flow.
- UI state: idle/loading/empty/success/error/cancelled.
- Keyboard/mouse interaction.
- Accessibility và usability.
- Main thread responsiveness.
- Test UI.

### Annotation Geometry
Issue cần mô tả:
- Coordinate system.
- Pixel/normalized conversion.
- XYXY/XYWH.
- Image orientation.
- Canvas zoom/pan transform.
- Polygon validity.
- Keypoint order/visibility.
- Undo/redo.
- Autosave.
- Geometry edge cases.

### YOLO
Phân biệt:
- Detection checkpoint.
- Segmentation checkpoint.
- Pose checkpoint.
- Model version.
- Class mapping.
- Confidence/NMS config.
- Output normalization.
- Device selection.
Không gom ba task vào một issue nếu acceptance criteria khác nhau.

### SAM 2 / SAM 3
Issue cần nêu:
- Prompt source: box/point/mask/other supported prompt.
- Input/output mask.
- Detector integration nếu có.
- Model/checkpoint/license.
- Refinement interaction.
- Performance/memory.
- Human review.
Không tuyên bố SAM tự sinh class label nếu pipeline không có classifier/detector.

### DINOv2 / DINOv3
Issue chỉ dành cho use case đã xác định:
- Embedding extraction.
- Similarity search.
- Duplicate/near-duplicate discovery.
- Clustering/visual search.
- Feature-assisted review.
Không tạo issue “DINO object detection” nếu không có detection head/pipeline được chứng minh.

### Hardware & Compute Routing
Issue phải xác định:
- Hardware probe.
- Capability profile.
- Local CPU/GPU.
- Model-task compatibility.
- Memory limits.
- OOM recovery.
- User override.
- Router explanation.
Không dùng ngưỡng VRAM chung không có benchmark.

### Kaggle CLI Remote Worker
Issue phải xác định:
- CLI setup/validation.
- Credential security.
- Input manifest/package.
- User consent before transfer.
- Upload/sync strategy.
- Remote job creation/execution.
- Job state/log.
- Retry/resume/session expiry.
- Output artifact transfer.
- Checksum/schema validation.
- Import result.
- Mock-based tests.
Không đặt token trong issue hoặc repo. Không giả định GPU quota cố định. Không thiết kế remote Kaggle như giao diện desktop thứ hai.

### Import/Export
Tách format/task phù hợp:
- YOLO detect.
- YOLO seg.
- YOLO pose.
- COCO instances/keypoints.
- Pascal VOC.
Acceptance phải có round-trip tests và class mapping.

### Data Quality
- Missing labels.
- Invalid geometry.
- Class ID mismatch.
- Keypoint mismatch.
- Empty annotations.
- Duplicate data.
- Dataset distribution.
- Review status.
- Export preflight.

## 7. PHASE 4 — ISSUE TITLE TAXONOMY

Dùng title mô tả outcome:
- [EPIC] ...
- [DESKTOP] ...
- [UI] ...
- [DATASET] ...
- [ANNOTATION] ...
- [BBOX] ...
- [SEG] ...
- [POSE] ...
- [MODEL] ...
- [YOLO] ...
- [SAM] ...
- [DINO] ...
- [INFERENCE] ...
- [HARDWARE] ...
- [COMPUTE] ...
- [KAGGLE] ...
- [IMPORT] ...
- [EXPORT] ...
- [VALIDATION] ...
- [TEST] ...
- [PACKAGING] ...
- [SECURITY] ...
- [DOCS] ...
- [BUG] ...
- [SPIKE] ...

Chỉ dùng prefix cần thiết. Title phải mô tả kết quả, không phải hoạt động mơ hồ.

## 8. PHASE 5 — ISSUE BODY TEMPLATE

Mỗi issue phải có:

# [Title]

## 1. Context
Tại sao issue tồn tại và giá trị đối với sản phẩm.

## 2. Objective
Một kết quả chính duy nhất.

## 3. Source of Truth
- Document:
- Section/Requirement ID:
- ADR:
- Parent Epic:

## 4. Scope
### In Scope
### Out of Scope

## 5. Technical Requirements
- Behavior.
- Inputs/outputs.
- Interface.
- State handling.
- Error handling.
- Logging.
- Performance.
- Security/privacy.
- Compatibility.

## 6. UX / Workflow (nếu liên quan)
- Entry point.
- User action.
- Expected UI state.
- Success/error/cancel behavior.

## 7. Implementation Plan
Các bước có thể thực hiện và review.

## 8. Deliverables
Source, tests, config, docs, screenshots/logs nếu phù hợp.

## 9. Acceptance Criteria
Checklist cụ thể, đo được, test được.
Không dùng “hoạt động tốt”, “nhanh”, “chính xác cao” nếu không có cách đo.

## 10. Verification
Unit/integration/UI/model/performance/manual tests.

## 11. Dependencies
Depends on / Blocks / Related.
Không tự bịa issue number.

## 12. Risks & Edge Cases
Các trường hợp lỗi và giới hạn.

## 13. Documentation Updates
Tài liệu phải cập nhật.

## 14. Definition of Done
Acceptance pass, tests pass, review, docs, no blocking issue.

## 15. Completion Evidence
PR, test report, screenshot, benchmark, model logs, export sample.

## 9. PHASE 6 — MILESTONES ĐỀ XUẤT

Điều chỉnh theo roadmap thực tế. Gợi ý:

M0 — Product Requirements & Technical Decisions.
M1 — Desktop Foundation.
M2 — Dataset & Project Management.
M3 — Annotation Core & BBox MVP.
M4 — SEG & POSE Workflows.
M5 — Local YOLO Auto-labeling.
M6 — SAM Model Integration.
M7 — Hardware Detection & Compute Router.
M8 — Kaggle CLI Remote Inference.
M9 — Review, Validation & Import/Export.
M10 — Performance, Recovery & Packaging.
M11 — Release & Handover.

Mỗi milestone:
- Goal.
- Included issues.
- Entry criteria.
- Exit criteria.
- Dependencies.
- Risks.
- Evidence.

Không đặt ngày nếu chưa được cung cấp.

## 10. PHASE 7 — LABEL GOVERNANCE

Kiểm tra labels hiện có trước.

Đề xuất taxonomy tối thiểu:
Type:
- type:epic
- type:feature
- type:bug
- type:research
- type:test
- type:documentation
- type:maintenance

Priority:
- priority:critical
- priority:high
- priority:medium
- priority:low

Area:
- area:desktop
- area:ui
- area:dataset
- area:annotation
- area:model
- area:inference
- area:hardware
- area:remote-compute
- area:import-export
- area:testing
- area:packaging
- area:security
- area:documentation

Status:
- status:ready
- status:in-progress
- status:blocked
- status:needs-review

Chỉ tạo labels cần dùng. Không duplicate.

## 11. PHASE 8 — DEPENDENCY & CRITICAL PATH

Xác định:
- Blocking dependencies.
- Soft dependencies.
- Parallelizable work.
- Critical path.
- ADR blockers.
- Model/license blockers.
- Kaggle access/credential blockers.
- Hardware-specific blockers.
- UI/core integration dependencies.

Không tạo circular dependency.

## 12. PHASE 9 — REPOSITORY GOVERNANCE

Nếu phù hợp, tạo:
.github/ISSUE_TEMPLATE/
- config.yml
- bug_report.yml
- feature_request.yml
- model_integration.yml
- annotation_task.yml
- infrastructure_task.yml

docs/06-ke-hoach/project-management/
- ISSUE_GOVERNANCE.md
- ISSUE_TAXONOMY.md
- MILESTONE_PLAN.md
- REQUIREMENTS_TRACEABILITY.md
- IMPLEMENTATION_ROADMAP.md

issue-drafts/
- README.md
- issues/

Không tạo file rỗng hoặc trùng lặp.

## 13. PHASE 10 — QUALITY AUDIT

Kiểm tra:
- Requirement coverage.
- Duplicate issues.
- Acceptance criteria.
- Correct dependencies.
- Correct milestone.
- Model task compatibility.
- Annotation schema traceability.
- Local/remote compute boundaries.
- Kaggle credential security.
- Import/export tests.
- UI responsiveness requirements.
- Packaging/release tasks.

Báo cáo số lượng thực tế:
- Requirements found/covered.
- Proposed issues.
- Created issues.
- Clarification issues.
- Duplicates.
- Missing acceptance criteria.
- Dependency conflicts.
- Remaining blockers.

Không tự tạo số liệu.

## 14. FINAL DELIVERABLES

A. Project understanding report.
B. Requirements traceability matrix.
C. Epic/work breakdown.
D. Issue catalog.
E. Full issue bodies.
F. Milestone plan.
G. Label catalog.
H. Dependency/critical path.
I. Repository governance files.
J. Quality audit.
K. GitHub execution report (nếu có).
L. First Ready issue to implement.

## 15. START EXECUTION

Bắt đầu bằng repository/documentation inventory. Chưa tạo issue ngay.

Trước tiên hãy trình bày:
1. Project understanding.
2. Requirements matrix.
3. Proposed Epics.
4. Proposed milestones.
5. Labels.
6. Dependencies.
7. Unresolved decisions.
8. Issue creation plan.

Nếu có GitHub write access, chỉ thực hiện sau khi đã kiểm tra duplicate và có approval theo yêu cầu. Nếu không, tạo draft local và ghi rõ trạng thái.

Mục tiêu cuối: một developer mới có thể chọn issue Ready, hiểu context, implement, test, review và trace ngược về yêu cầu mà không cần hỏi lại các thông tin đã có trong tài liệu.
```

---
