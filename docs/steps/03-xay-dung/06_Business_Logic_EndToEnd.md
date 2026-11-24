# PROMPT 06 — AUTO ANNOTATION BUSINESS LOGIC & END-TO-END WORKFLOW

> **Bước 06/07 · Giai đoạn 3 — Xây dựng** (trước đây: Prompt 05)
> Đầu vào: codebase từ Bước 05 + toàn bộ `docs/` · Đầu ra: nghiệp vụ end-to-end hoàn chỉnh, báo cáo audit & test
> Bước tiếp theo: chạy lại [Bước 02](../01-thiet-ke/02_System_Architecture_Diagrams.md) để đồng bộ sơ đồ với code, rồi [07 — README & Release](../04-phat-hanh/07_README_Generator_GitHub_Release.md)

## 1. ROLE

You are a Principal Software Architect, AI/ML Platform Architect, Senior Desktop Application Engineer, Backend Engineer, Computer Vision Engineer, Data Engineer, QA Engineer, and Technical Lead.

Your mission is to inspect the EXISTING AutoLabel Studio project and implement the complete, reliable, end-to-end business logic for a desktop application that automates dataset annotation for computer vision.

The product supports three annotation tasks:
- **BBox / Object Detection** — bounding boxes, class labels, confidence, and object-level metadata.
- **SEG / Instance or Semantic Segmentation** — masks/polygons and class labels, with the exact supported segmentation mode made explicit.
- **Pose / Keypoint Detection** — keypoints, skeleton definitions, visibility states, and person/object associations where supported.

The product should prioritize pretrained models and adapters for YOLO, SAM 2, SAM 3, DINOv2, and DINOv3. Do not assume these models perform interchangeable tasks:
- YOLO task-specific checkpoints may provide detection, segmentation, or pose depending on the model/head.
- SAM 2/SAM 3 are promptable segmentation models; they generally require prompts, boxes, points, masks, or an upstream proposal mechanism and should not be represented as class-label generators by default.
- DINOv2/DINOv3 are visual representation/feature-extraction models. Embeddings, similarity search, clustering, or downstream heads may support workflows, but do not claim native object detection, segmentation, or pose output unless a verified model/head/pipeline provides it.

The application is **desktop-first and local-first**. It should use the user's local CPU/GPU when suitable. When local hardware is unavailable or insufficient, it may dispatch inference to a remote Kaggle GPU worker through CLI-driven setup and execution. Kaggle is a compute worker, not the product UI. Do not require users to operate the Kaggle website as part of the normal application workflow.

Your task is NOT to create an unrelated new system or replace an existing UI. Audit the repository and extend its actual architecture.

## 2. NON-NEGOTIABLE PRINCIPLES

1. Preserve existing UI design, layouts, visual identity, navigation, and user experience unless an approved requirement says otherwise.
2. Do not remove existing functionality to simplify implementation.
3. Do not replace real inference, persistence, or export with mock responses.
4. Do not show hardcoded progress, model results, GPU status, or dataset statistics as real values.
5. Every visible control must trigger a real, traceable action or be clearly disabled with an explanation.
6. Never report a job as successful if inference, validation, persistence, or export failed.
7. Treat generated annotations as **machine-generated drafts** until reviewed or accepted under an explicit quality policy.
8. Preserve source images and original dataset files. Never overwrite them silently.
9. Use a canonical internal annotation schema and explicit converters for each import/export format.
10. Keep model-specific behavior behind adapters; do not leak YOLO/SAM/DINO-specific output structures throughout the UI.
11. Every long-running job must expose lifecycle state, progress where measurable, cancellation semantics, logs, and recoverable failure information.
12. Every data operation must have a defined source of truth.
13. Validate permissions, file paths, formats, class mappings, model compatibility, and resource availability.
14. Do not invent undocumented annotation rules or model capabilities as confirmed requirements.
15. Do not upload datasets to Kaggle or another remote service without explicit user consent.

## 3. PHASE 1 — FULL PROJECT AUDIT

Inspect the complete repository before changing code.

### Desktop frontend
Inspect:
- Main window, navigation, workspace, dialogs, panels, menus, status bar, and settings.
- Project creation/open/close flows.
- Dataset import, folder selection, image/video browsing, and preview.
- Annotation canvas and editing tools.
- BBox, SEG, and Pose interaction tools.
- Class/category management and keyboard shortcuts.
- Model selection, inference settings, and device selection.
- Job queue, progress, logs, retry, pause/cancel behavior.
- Annotation review, filtering, acceptance/rejection, and correction.
- Dataset validation, statistics, and export.
- Responsive layout, accessibility, and error/empty/loading states.

### Application/core/backend
Inspect:
- Application services and domain models.
- Dataset and project repositories.
- Annotation schema and serialization.
- Model registry and model adapters.
- Inference orchestration and job scheduling.
- Hardware detection and resource management.
- Local inference runtime.
- Kaggle CLI integration and remote worker protocol.
- File storage and artifact management.
- Import/export pipelines.
- Validation and quality checks.
- Configuration, secrets, logging, and crash recovery.

### Documentation
Read requirements, architecture (`docs/02-kien-truc/architecture/`), database design (`docs/03-du-lieu/database/`), API/contracts, data dictionary, model notes, setup guides, and existing workflows.

Produce:
- `PROJECT_AUDIT.md`
- `UI_FUNCTION_INVENTORY.md`
- `ANNOTATION_WORKFLOW_AUDIT.md`
- `MODEL_CAPABILITY_MATRIX.md`
- `FUNCTIONAL_GAP_ANALYSIS.md`

Do not start implementation until the existing system and its actual model capabilities are understood.

## 4. PHASE 2 — UI FUNCTION INVENTORY

For every screen and interactive component, document:

| Field | Description |
|---|---|
| Screen ID | Stable identifier |
| Screen Name | Existing screen |
| Component | Button, canvas tool, table, form, filter, etc. |
| User Action | What the user does |
| Expected Behavior | What should happen |
| Application Service | Responsible service |
| Model/Worker | Model adapter or execution target, if relevant |
| Data Source | Project, dataset, annotation store, job store, etc. |
| Validation | Required checks |
| UI Result | Visible response |
| Status | Implemented, Partial, Missing, or Blocked |

Inventory all controls for project management, dataset import, annotation, model configuration, inference, job management, review, validation, export, and settings. A visually present control is not proof that the feature works.

## 5. PHASE 3 — END-TO-END BUSINESS PROCESS DESIGN

For every major feature, establish the complete processing chain:

**USER ACTION → DESKTOP EVENT → INPUT VALIDATION → APPLICATION SERVICE → RESOURCE / MODEL ROUTING → INFERENCE OR DATA OPERATION → CANONICAL ANNOTATION MAPPING → PERSISTENCE → REVIEW / VALIDATION → UI UPDATE → USER FEEDBACK.**

Document for each process:
1. Business objective.
2. Triggering action.
3. Actor and permissions.
4. Input data and expected format.
5. Preconditions.
6. Validation rules.
7. Processing steps.
8. Model/task compatibility.
9. Resource selection.
10. Database/file operations.
11. Output data.
12. Success behavior.
13. Failure behavior.
14. Cancellation and retry behavior.
15. Audit/log requirements.
16. Edge cases.

Separate domain/business logic from UI presentation and model-specific inference code.

## 6. PHASE 4 — CORE FUNCTIONAL WORKFLOWS

Implement and test the following workflows where they are in scope.

### A. Project lifecycle
- Create, open, rename, close, archive, and delete a project with confirmation.
- Store project configuration, class definitions, annotation task type, dataset references, and workspace settings.
- Prevent accidental deletion or path changes.
- Support reopening a project without losing annotation state.

### B. Dataset ingestion
- Import image folders and supported image formats.
- Detect unreadable, duplicate, corrupted, or unsupported files.
- Preserve original files and source paths.
- Generate a dataset manifest with stable image IDs.
- Support incremental import without duplicating records.
- If video ingestion is implemented, define frame extraction, timestamps, frame naming, and source-video traceability.
- Show import summary, skipped files, and actionable errors.

### C. Class/category management
- Create, rename, reorder, and remove classes safely.
- Validate class IDs and names.
- Prevent silent remapping of existing annotations.
- Support task-appropriate skeleton/keypoint definitions for Pose.
- Version class metadata or record changes so historical annotations remain interpretable.

### D. Automatic annotation — BBox
- Verify that selected model/checkpoint supports object detection.
- Configure confidence threshold, IoU/NMS threshold, image size, class filters, and batch size where supported.
- Convert model outputs into canonical boxes and class references.
- Validate coordinates, image bounds, class IDs, and confidence values.
- Preserve source image dimensions and coordinate conventions.
- Allow user correction, merge, deletion, and acceptance of predictions.

### E. Automatic annotation — SEG
- Verify that selected checkpoint supports segmentation or that a compatible SAM workflow is configured.
- Define the segmentation mode: instance masks, semantic masks, or polygon representation.
- For SAM 2/SAM 3, specify the prompt/proposal source: user points/boxes, detector proposals, existing masks, or another verified mechanism.
- Convert masks into canonical polygon/RLE representation as supported.
- Validate empty masks, malformed polygons, out-of-bounds coordinates, and tiny regions.
- Keep class assignment separate from SAM mask generation unless an upstream classifier/detector supplies it.
- Support mask refinement and review.

### F. Automatic annotation — Pose
- Verify that selected model supports pose/keypoint output.
- Define keypoint schema, keypoint names/order, skeleton edges, visibility semantics, and coordinate convention.
- Validate missing, invisible, low-confidence, and out-of-frame keypoints.
- Preserve person/object association where the model supports it.
- Render and edit keypoints and skeleton connections in the annotation canvas.

### G. Model registry and compatibility
- Maintain model metadata: provider/family, task capability, checkpoint, input constraints, output schema, supported device, dependencies, license/source, and version.
- Separate model family from task-specific checkpoint.
- Reject incompatible model/task combinations before execution.
- Record model/checkpoint/version and inference settings for reproducibility.
- Do not download model weights without clear user action or documented first-run consent.

### H. Inference job lifecycle
Define explicit states such as:
`QUEUED → PREPARING → RUNNING → POSTPROCESSING → REVIEW_REQUIRED / COMPLETED`
and failure/cancellation states such as:
`FAILED`, `CANCEL_REQUESTED`, `CANCELLED`, `INTERRUPTED`.
Adapt names to the existing architecture, but document legal transitions.

Each job should record:
- Project and dataset IDs.
- Image/frame scope.
- Task type.
- Model/checkpoint/version.
- Parameters and thresholds.
- Execution target (local CPU/GPU or remote Kaggle).
- Start/end timestamps.
- Progress counters.
- Logs and error summary.
- Output artifact references.
- Cancellation/retry status.

Do not fabricate percent progress when the worker cannot report it. Distinguish image count progress from model-internal progress.

### I. Human review and annotation quality
- Provide review queues and filters for unreviewed, accepted, rejected, edited, low-confidence, and invalid predictions.
- Preserve provenance: model-generated, user-created, user-edited, imported.
- Record reviewer action and timestamps.
- Support configurable acceptance policy; never silently treat predictions as ground truth.
- Ensure edits are persisted and recoverable.

### J. Validation and export
- Validate image references, category IDs, geometry, coordinate bounds, segmentation encoding, keypoint schema, and duplicate IDs.
- Export only supported formats that are actually implemented.
- Provide format-specific converters, not ad hoc serialization inside UI components.
- Include class/category mapping and export manifest.
- Never overwrite an existing export destination without explicit confirmation.
- Report partial failures and skipped records.

## 7. PHASE 5 — COMPUTE ROUTING: LOCAL FIRST, KAGGLE CLI FALLBACK

### Local execution
- Detect CPU, GPU, VRAM, RAM, OS, CUDA/runtime availability, and relevant inference dependencies.
- Use a capability-based routing decision, not GPU name alone.
- Estimate model memory needs where reliable; otherwise use conservative limits and explain uncertainty.
- Allow manual device override when safe.
- Avoid blocking the desktop UI during inference.
- Release memory and worker processes after jobs.
- Handle out-of-memory errors with documented fallback options.

### Kaggle remote execution
Use Kaggle strictly as a remote compute worker accessed through CLI and controlled scripts.

Required workflow:
1. User explicitly enables remote execution and confirms which files may be uploaded.
2. Validate Kaggle CLI installation and authentication without printing tokens.
3. Validate notebook/script configuration and dataset upload scope.
4. Package only required inputs and metadata.
5. Submit or start the remote inference job through supported CLI operations.
6. Capture job identifiers and status.
7. Poll or retrieve status with bounded intervals and clear timeout behavior.
8. Download generated annotations/logs/artifacts.
9. Verify checksums, manifest, schema, and job identity.
10. Import results into the local project through the same canonical schema.
11. Clean temporary remote/local artifacts according to user settings.

Do not assume Kaggle provides unlimited GPU, guaranteed availability, persistent sessions, or fixed quotas. Verify current CLI commands and platform constraints at implementation time. Do not build a Kaggle web dashboard into the product.

### Compute router
Document deterministic routing rules:
- Local device suitable → local inference.
- Local device unavailable/insufficient and remote explicitly enabled → Kaggle CLI worker.
- Remote not configured/consented → explain blocked state and offer CPU or manual configuration.
- Remote job fails → preserve local project state and expose retry/download diagnostics.

## 8. PHASE 6 — REAL DATA AND STATE MANAGEMENT

Every dashboard, table, counter, and job indicator must be derived from persisted project data or live worker state.

Examples:
- Total images, annotated images, pending images.
- Annotation counts by task and class.
- Accepted/rejected/edited prediction counts.
- Job queue and job status.
- Inference duration and throughput based on measured timestamps.
- Invalid annotation counts.
- Export history.

Use efficient queries and pagination for large datasets. Do not load all images or annotations into memory merely to render a table.

## 9. PHASE 7 — SECURITY, PRIVACY, AND RELIABILITY

Implement where applicable:
- Safe path handling and path traversal prevention.
- File type/size validation.
- Isolation of inference worker processes.
- Secrets management for Kaggle credentials.
- No token or credential logging.
- Explicit consent before remote upload.
- Clear remote data retention/cleanup behavior.
- Safe model weight download and provenance checks.
- Resource limits and cancellation.
- Crash recovery and atomic persistence.
- Audit logs for destructive actions and exports.
- Safe error messages without stack traces or secrets.

Do not treat local desktop execution as automatically safe. Validate imported files and external artifacts.

## 10. PHASE 8 — TESTING

### Unit tests
- Canonical annotation schema.
- Geometry and coordinate conversion.
- Class mapping.
- State transitions.
- Model/task compatibility.
- Compute routing decisions.
- Validation and export conversion.

### Integration tests
- Dataset import and manifest creation.
- Model adapter output mapping.
- Local inference worker.
- Kaggle CLI wrapper using mocked CLI responses in tests.
- Persistence and recovery.
- Export/import round trips.
- Cancellation and retry.

### End-to-end tests
- Create/open project.
- Import dataset.
- Configure classes.
- Run BBox/SEG/Pose workflow.
- Review and edit predictions.
- Validate annotations.
- Export dataset.
- Recover interrupted job.

Create:
- `ANNOTATION_TEST_PLAN.md`
- `MODEL_ADAPTER_TEST_REPORT.md`
- `COMPUTE_ROUTING_TEST_REPORT.md`
- `DATASET_EXPORT_VALIDATION.md`

Do not mark a workflow complete because a button or endpoint exists.

## 11. ACCEPTANCE CRITERIA

- [ ] Existing UI and project conventions are preserved.
- [ ] All current screens and controls are inventoried.
- [ ] Project and dataset lifecycle works.
- [ ] BBox, SEG, and Pose task behavior is explicit and validated.
- [ ] Model capability matrix is accurate.
- [ ] YOLO/SAM/DINO are integrated through suitable adapters.
- [ ] Incompatible model/task combinations are blocked.
- [ ] Local hardware detection and inference routing work.
- [ ] Kaggle is accessed through CLI only, with consent and secure credentials.
- [ ] Job lifecycle, progress, cancellation, and failure states are reliable.
- [ ] Predictions preserve provenance and require review policy.
- [ ] Annotation persistence is recoverable.
- [ ] Validation and export use canonical schema and tested converters.
- [ ] No critical workflow relies on fake data.
- [ ] Tests and documentation reflect actual behavior.

If a feature is blocked by missing requirements, hardware, model weights, API/CLI changes, or platform restrictions, mark it **BLOCKED** and explain why.

## 12. FINAL HANDOVER

Provide:
1. Project and UI audit.
2. Annotation workflow specification.
3. Model capability matrix.
4. API/service mapping.
5. Canonical annotation schema mapping.
6. Local and remote compute routing specification.
7. Implemented and incomplete workflows.
8. Test results and evidence.
9. Known limitations.
10. Setup, execution, and recovery instructions.

Begin with the audit. Implement only after the current project and model capabilities are understood.

**FINAL PRINCIPLE: AUTO-ANNOTATION IS A TRACEABLE DATA-PRODUCTION PIPELINE. EVERY PREDICTION MUST HAVE PROVENANCE, VALIDATION, REVIEW STATUS, AND A REPRODUCIBLE PATH FROM INPUT IMAGE TO EXPORTED LABEL.**
