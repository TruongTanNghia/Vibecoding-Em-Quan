# PROMPT 02 — SYSTEM ARCHITECTURE & TECHNICAL DIAGRAMS

> **Bước 02/07 · Giai đoạn 1 — Thiết kế & Tài liệu** (trước đây: Prompt 06)
> Đầu vào: `docs/` từ Bước 01 · Đầu ra: `docs/02-kien-truc/architecture/` (sơ đồ Mermaid + ADR)
> Ở giai đoạn thiết kế chưa có source code → mọi thành phần được đánh dấu **PROPOSED**. Chạy lại prompt này ở chế độ đồng bộ với code trước Bước 07.
> Bước tiếp theo: [03 — Database Architecture](03_Database_Architecture.md)

## 1. ROLE

You are a Principal Software Architect, Desktop Platform Architect, Computer Vision Systems Architect, AI/ML Infrastructure Architect, Data Flow Analyst, and Technical Documentation Engineer.

Your mission is to inspect the existing AutoLabel Studio repository and create a complete, technically accurate set of architecture diagrams for the desktop automatic annotation platform.

The product supports BBox, SEG, and Pose annotation; model families include task-specific YOLO checkpoints, SAM 2, SAM 3, DINOv2, and DINOv3 where their verified capabilities fit. It runs inference locally when possible and can use Kaggle as a CLI-controlled remote GPU worker when local hardware is unavailable or insufficient.

Base every diagram on actual source code, approved requirements, verified model capabilities, and implemented or explicitly proposed architecture. Do not invent components merely to make diagrams look enterprise-grade.

## 2. PRIMARY OBJECTIVES

Explain:
- What the desktop application does.
- How the desktop UI, application core, domain services, and storage interact.
- How datasets are imported, indexed, and managed.
- How BBox, SEG, and Pose annotation workflows operate.
- How model adapters normalize YOLO/SAM/DINO outputs.
- How local hardware is detected and selected.
- How Kaggle CLI remote execution is submitted, monitored, and retrieved.
- How annotations are reviewed, validated, persisted, and exported.
- How jobs recover from failures and cancellations.
- How project data and model artifacts are stored.
- How the application is packaged, installed, and updated if those features exist.

Preserve existing architecture unless a documented technical reason supports a change.

## 3. PHASE 1 — ARCHITECTURE DISCOVERY

Inspect:
- Desktop framework and UI architecture.
- Application entry points and lifecycle.
- Frontend/view-model/presenter layers, if present.
- Domain models and application services.
- Dataset and annotation repositories.
- Model registry and adapters.
- Inference workers and process boundaries.
- Hardware detection and compute router.
- Kaggle CLI integration and remote worker scripts.
- File system layout and artifact storage.
- Import/export converters.
- Job queue and state persistence.
- Logging, settings, secrets, and crash recovery.
- Build, packaging, installer, and deployment configuration.
- Existing documentation and diagrams.

Produce an architecture inventory:

| Field | Description |
|---|---|
| Component ID | Stable identifier |
| Component Name | Actual component |
| Layer | UI, domain, application, inference, storage, infrastructure |
| Responsibility | What it does |
| Technology | Verified framework/tool |
| Input | Incoming data |
| Output | Produced data |
| Dependencies | Connected components |
| Execution Location | Desktop, local worker, Kaggle worker |
| Status | Existing, Partial, Proposed |

Clearly distinguish implemented, partial, and proposed components.

## 4. PHASE 2 — REQUIRED DIAGRAMS

Generate the following diagrams where applicable.

### DIAGRAM 01 — SYSTEM CONTEXT
Show:
- Desktop user / annotator / project administrator.
- AutoLabel Studio system boundary.
- Local filesystem and local hardware.
- Model weight sources or model repositories, only if used.
- Kaggle platform/CLI as an external compute service.
- Optional external dataset sources, only if integrated.
- Export destinations and supported dataset consumers.
- Main data and control interactions.

Explain what is inside and outside the product boundary. Do not portray Kaggle as the application's frontend or as a mandatory dependency.

### DIAGRAM 02 — HIGH-LEVEL DESKTOP ARCHITECTURE
Show the actual architecture, using a suitable decomposition such as:

**USER → DESKTOP UI → APPLICATION / DOMAIN SERVICES → MODEL & COMPUTE ORCHESTRATION → LOCAL OR REMOTE WORKER → ANNOTATION NORMALIZATION → PROJECT STORAGE → REVIEW / VALIDATION → EXPORT.**

Include only verified or explicitly proposed components:
- Project manager.
- Dataset manager/indexer.
- Annotation canvas.
- Class/category manager.
- Model registry.
- Model adapters.
- Inference orchestrator.
- Local CPU/GPU worker.
- Kaggle CLI adapter/remote worker.
- Job queue and status store.
- Canonical annotation store.
- File/artifact storage.
- Validator and exporter.
- Logging and monitoring.

Show synchronous calls versus asynchronous job execution and process boundaries.

### DIAGRAM 03 — APPLICATION COMPONENT DIAGRAM
Break down:
- Desktop UI modules.
- Project and dataset domain services.
- Annotation domain and geometry services.
- Model registry and adapters.
- Inference orchestration.
- Hardware detection and compute routing.
- Kaggle CLI integration.
- Job lifecycle manager.
- Persistence and artifact services.
- Review, validation, and export modules.

Explain responsibilities and avoid simply reproducing directory names.

### DIAGRAM 04 — PROJECT AND DATASET LIFECYCLE
Show project creation/opening, dataset import, file validation, stable image IDs, manifest generation, class configuration, incremental import, project save, close, and recovery.

Include duplicate, corrupt, unsupported, missing-file, and permission-error branches.

### DIAGRAM 05 — BBOX AUTO-ANNOTATION FLOW
Show:
- User selects dataset scope and BBox task.
- Model/checkpoint compatibility check.
- Parameter validation.
- Compute router decision.
- Local inference or Kaggle CLI remote execution.
- Model output normalization.
- Coordinate and class validation.
- Prediction persistence.
- Review/edit/accept/reject.
- Final export eligibility.

### DIAGRAM 06 — SEG AUTO-ANNOTATION FLOW
Show:
- Segmentation-capable checkpoint or SAM workflow selection.
- Prompt/proposal source for SAM (points, boxes, masks, detector proposals, or other verified input).
- Mask inference/refinement.
- Optional class assignment from a separate detector/classifier.
- Mask-to-canonical representation conversion.
- Polygon/RLE and geometry validation.
- Review and export.

Explicitly show that SAM mask generation does not automatically imply semantic class identification. Distinguish instance versus semantic segmentation if both are supported.

### DIAGRAM 07 — POSE AUTO-ANNOTATION FLOW
Show:
- Pose-capable checkpoint selection.
- Keypoint schema/skeleton compatibility.
- Local/remote inference.
- Keypoint and confidence normalization.
- Visibility and coordinate validation.
- Person/object association if supported.
- Review and editing.
- Export conversion.

### DIAGRAM 08 — MODEL ADAPTER AND CAPABILITY MATRIX
Show model-family adapters and task capability boundaries:
- YOLO detection, segmentation, or pose only for suitable task-specific model/checkpoint.
- SAM 2/SAM 3 promptable segmentation with explicit prompt/proposal input.
- DINOv2/DINOv3 feature extraction/embeddings/similarity/clustering or verified downstream head.
- Any downstream detector/segmenter/head as a separately named component.

Show model metadata, checkpoint/version, preprocessing, inference, postprocessing, canonical output mapping, and error boundary. Never imply all models support all tasks.

### DIAGRAM 09 — LOCAL COMPUTE ROUTING
Show:
- OS/runtime/dependency checks.
- CPU/GPU detection.
- CUDA/runtime availability.
- VRAM/RAM checks.
- Model/task requirements.
- User override where supported.
- Local suitability decision.
- Local worker launch.
- Out-of-memory/runtime failure handling.
- Remote eligibility and explicit consent gate.

Do not infer compute suitability from GPU name alone.

### DIAGRAM 10 — KAGGLE CLI REMOTE EXECUTION SEQUENCE
Show:
- User opt-in and upload scope confirmation.
- CLI/authentication validation.
- Safe packaging of selected data.
- Remote job submission.
- Job ID capture.
- Status polling or supported status retrieval.
- Timeout/cancel/failure handling.
- Artifact/log download.
- Integrity and schema validation.
- Import into local project.
- Temporary artifact cleanup.

Mark Kaggle as external infrastructure. Never show a Kaggle web UI as the product workflow. Never draw credentials or tokens as data payloads.

### DIAGRAM 11 — ANNOTATION DATA FLOW
Show:
- Source images/video frames.
- Dataset manifest.
- Class and task configuration.
- Model/checkpoint and parameters.
- Raw model output.
- Canonical annotation object.
- Provenance metadata.
- Review state.
- Validation result.
- Export artifact.

Identify trust boundaries, temporary artifacts, persistent records, and remote upload boundaries.

### DIAGRAM 12 — CANONICAL DATA MODEL / ER DIAGRAM
Show project, dataset, image/frame, category/class, annotation, geometry, keypoint schema, model, inference job, job artifact, review event, validation report, export job, and audit event only if they exist or are approved for implementation.

Use actual schema when available. Label proposed entities **PROPOSED**. Do not invent implemented columns.

### DIAGRAM 13 — JOB STATE TRANSITION
Show allowed inference job states, including queued, preparing, running, post-processing, review-required/completed, failed, cancel-requested, cancelled, and interrupted as applicable. Include transition guards and retry behavior.

### DIAGRAM 14 — DEPLOYMENT / PACKAGING DIAGRAM
Show desktop executable, local runtime, local model weights, local project storage, worker processes, optional Kaggle CLI dependency, configuration, and update mechanism only if present. Include OS support only when verified.

Do not invent cloud-hosted backend infrastructure if the product is desktop/local-first.

### DIAGRAM 15 — ERROR HANDLING AND RECOVERY
Show:
- Dataset read/permission errors.
- Invalid/corrupt images.
- Unsupported model/task.
- Missing weights/dependencies.
- Local OOM or runtime crash.
- Kaggle authentication/submission/timeout errors.
- Remote artifact corruption.
- Annotation validation failure.
- Persistence/export failure.
- Retry, fallback, rollback, recovery, and user notification.

### DIAGRAM 16 — PERFORMANCE AND SCALABILITY
Show relevant strategies:
- Batch inference.
- Bounded worker concurrency.
- Memory-aware image loading.
- Lazy image/annotation loading.
- Incremental dataset indexing.
- Caching where justified.
- Streaming progress.
- Disk-backed artifacts.
- Database indexes and pagination.
- Local/remote workload routing.

Clearly distinguish current behavior from future proposals.

## 5. PHASE 3 — DIAGRAM STANDARDS

Use Mermaid as the default diagram-as-code format.

Prefer:
- `flowchart` for workflows and architecture.
- `sequenceDiagram` for component interactions.
- `erDiagram` for database relationships.
- `stateDiagram-v2` for job and lifecycle states.
- C4 notation only if the renderer supports it.

Each diagram must include:
1. Diagram ID.
2. Title.
3. Objective.
4. Scope.
5. Actors/components.
6. Flow explanation.
7. Technical assumptions.
8. Related code/documentation.
9. Implementation status.

Use consistent names across all diagrams. Label protocols and boundaries only when verified. Break large diagrams into smaller diagrams with clear boundaries.

## 6. PHASE 4 — TRACEABILITY TO IMPLEMENTATION

Map diagrams to real project artifacts:

**DESKTOP UI COMPONENT → APPLICATION SERVICE → DOMAIN OPERATION → MODEL ADAPTER / WORKER → ANNOTATION MAPPER → REPOSITORY / FILE STORE → VALIDATOR / EXPORTER.**

Include:
- Source file paths.
- Module names.
- Class/function names.
- Data models.
- Model/checkpoint identifiers.
- CLI commands/scripts.
- Storage locations.
- Related requirements and workflows.

If a component exists only in a proposal, mark it **PROPOSED**.

## 7. PHASE 5 — DOCUMENTATION STRUCTURE

Preserve existing documentation organization. If appropriate, create:

```text
docs/02-kien-truc/
  architecture/
    README.md
    SYSTEM_CONTEXT.md
    HIGH_LEVEL_DESKTOP_ARCHITECTURE.md
    APPLICATION_COMPONENTS.md
    PROJECT_DATASET_LIFECYCLE.md
    BBOX_WORKFLOW.md
    SEG_WORKFLOW.md
    POSE_WORKFLOW.md
    MODEL_ADAPTER_ARCHITECTURE.md
    LOCAL_COMPUTE_ROUTING.md
    KAGGLE_CLI_REMOTE_EXECUTION.md
    ANNOTATION_DATA_FLOW.md
    CANONICAL_DATA_MODEL.md
    JOB_STATE_MACHINE.md
    DEPLOYMENT_AND_PACKAGING.md
    ERROR_RECOVERY.md
    PERFORMANCE_ARCHITECTURE.md
    ARCHITECTURE_DECISIONS.md
    diagrams/
      system_context.mmd
      desktop_architecture.mmd
      component_architecture.mmd
      bbox_workflow.mmd
      segmentation_workflow.mmd
      pose_workflow.mmd
      local_compute_routing.mmd
      kaggle_remote_sequence.mmd
      annotation_data_flow.mmd
      canonical_data_model.mmd
      job_state_machine.mmd
```

Only create relevant files. Do not duplicate existing diagrams without review.

## 8. PHASE 6 — ARCHITECTURE DECISION RECORDS

For major decisions, create ADRs with:
- ADR identifier and title.
- Status.
- Context.
- Decision.
- Alternatives.
- Consequences.
- Risks.
- Related diagrams and requirements.

Potential decisions include:
- Desktop framework and process model.
- Canonical annotation representation.
- Local inference runtime.
- Model adapter boundary.
- SAM prompt/proposal strategy.
- DINO downstream-head strategy.
- Kaggle CLI worker protocol.
- Local project storage/database.
- Artifact retention and remote cleanup.

Do not mark proposals as approved decisions.

## 9. PHASE 7 — DIAGRAM VALIDATION

Validate:
- Mermaid syntax.
- Missing/undefined nodes.
- Direction and relationship accuracy.
- Naming consistency.
- Model/task capability accuracy.
- Local versus remote boundaries.
- Current versus proposed status.
- Contradictions with source code.
- Contradictions with Prompt 01 workflows.
- Contradictions with database schema and migrations (once Step 03 has produced `docs/03-du-lieu/database/`).
- Deployment/package accuracy.

Render and visually inspect diagrams when tools are available.

## 10. ACCEPTANCE CRITERIA

- [ ] System context is accurate.
- [ ] Desktop architecture and process boundaries are documented.
- [ ] Project/dataset lifecycle is documented.
- [ ] BBox, SEG, and Pose workflows are separate and accurate.
- [ ] YOLO/SAM/DINO capability boundaries are explicit.
- [ ] Local compute routing is documented.
- [ ] Kaggle CLI remote workflow is documented with consent/security boundaries.
- [ ] Annotation data flow and provenance are clear.
- [ ] ERD matches actual or clearly proposed schema.
- [ ] Job state machine includes failure/cancellation paths.
- [ ] Packaging/deployment reflects actual product.
- [ ] Error recovery and performance are documented.
- [ ] Every diagram maps to implementation or is labeled proposed.
- [ ] Mermaid source validates.

## 11. FINAL HANDOVER

Provide:
1. Architecture inventory.
2. Diagram catalog.
3. Mermaid source files.
4. Diagram explanations.
5. Component-to-code mapping.
6. Model capability and adapter mapping.
7. Local/remote compute architecture.
8. Canonical data model mapping.
9. ADR documents.
10. Diagram validation report.
11. Outstanding architecture decisions.

**FINAL PRINCIPLE: THE DIAGRAMS MUST DESCRIBE THE REAL AUTOLABEL STUDIO SYSTEM — INCLUDING MODEL LIMITATIONS, LOCAL/REMOTE EXECUTION BOUNDARIES, AND THE FULL PATH FROM SOURCE DATA TO REVIEWED EXPORTS.**
