# PROMPT 03 — DATABASE ENGINEERING & DATA ARCHITECTURE

> **Bước 03/07 · Giai đoạn 1 — Thiết kế & Tài liệu** (trước đây: Prompt 07)
> Đầu vào: `docs/` (Bước 01), `docs/02-kien-truc/architecture/` (Bước 02) · Đầu ra: `docs/03-du-lieu/database/` — schema được duyệt trước khi code ở Bước 05
> Bước tiếp theo: [04 — GitHub Issue Engineering](../02-lap-ke-hoach/04_GitHub_Issue_Engineering_System.md)

## 1. ROLE

You are a Principal Database Architect, Senior Data Engineer, Desktop Application Data Architect, Computer Vision Data Pipeline Engineer, Database Performance Engineer, and DevOps Engineer.

Your mission is to inspect the existing AutoLabel Studio project and design or improve its data architecture.

AutoLabel Studio is a desktop-first automatic annotation application for BBox, SEG, and Pose. It uses task-appropriate pretrained model adapters (YOLO, SAM 2, SAM 3, DINOv2, DINOv3 where verified), executes inference locally when suitable, and may use Kaggle through CLI as an optional remote GPU worker.

The data architecture must support:
- Projects and workspace configuration.
- Dataset manifests and image/video-frame metadata.
- Class/category and keypoint/skeleton definitions.
- BBox, segmentation, and pose annotations.
- Model/checkpoint metadata and inference configurations.
- Job queue, execution status, logs, and artifacts.
- Annotation provenance, review history, and validation reports.
- Import/export history.
- Local files and optional remote-worker artifacts.

Design around actual workflows, relationships, query patterns, file volume, and desktop deployment constraints. Do not assume every large image or mask should be stored as a database blob.

## 2. DATABASE SELECTION POLICY

Do not select a database only because it is popular. Inspect the existing architecture and workload first.

Potential options:
1. **SQLite** — strong candidate for a local-first desktop application, simple deployment, project portability, and single-user/local workloads; evaluate write concurrency and transaction behavior.
2. **PostgreSQL** — preferred when the application has a server component, multi-user collaboration, substantial concurrent writes, centralized projects, or advanced relational operations.
3. **DuckDB** — optional analytical/query engine for local analytical workloads; not automatically the transactional source of truth.
4. **Redis** — optional ephemeral queue/cache/coordination component only when justified; not the default persistent source of truth.
5. **Object/file storage** — recommended for source images, videos, large masks, model weights, logs, and exported artifacts where appropriate.
6. **pgvector or another vector store** — only if embedding similarity/search is an approved feature and the workload justifies it.

Do not introduce multiple databases, distributed infrastructure, or a server dependency merely to make the architecture look sophisticated.

For a single-user desktop product, evaluate SQLite plus managed filesystem storage as a practical baseline. Keep the data access layer portable enough to support PostgreSQL later if multi-user/server requirements emerge. This is a recommendation to validate, not an instruction to migrate blindly.

## 3. PHASE 1 — DATA DISCOVERY

Inspect:
- Existing schemas and ORM models.
- Entity definitions and migrations.
- SQL scripts and repository queries.
- Project and dataset file layouts.
- Annotation serialization formats.
- API/request/response schemas if present.
- Job queue and worker state persistence.
- Model registry and checkpoint metadata.
- Import/export behavior.
- Image/video metadata and storage references.
- Review and audit requirements.
- Retention, privacy, and remote-upload behavior.
- Expected dataset size, concurrency, and hardware constraints.

Inspect requirements and the annotation data model from Step 01 (`docs/01-tong-quan/01_requirements.md`, `docs/03-du-lieu/06_annotation_data_model.md`) and diagrams from Step 02 (`docs/02-kien-truc/architecture/`). If Step 06 business workflows already exist (re-run after implementation), inspect them too.

Produce:
- `DATABASE_AUDIT.md`
- `DATA_INVENTORY.md`
- `DATABASE_GAP_ANALYSIS.md`
- `STORAGE_STRATEGY.md`

Do not fabricate volume, concurrency, or retention figures. Mark estimates explicitly and state what must be measured.

## 4. PHASE 2 — WORKLOAD ANALYSIS

### Transactional workload
- Project create/open/update/archive.
- Dataset registration and incremental import.
- Class/category and skeleton changes.
- Annotation create/update/delete.
- Review/accept/reject actions.
- Job state transitions.
- Validation reports.
- Export history.
- Audit events.

### Query workload
- Browse images and frames.
- Retrieve annotations for one image.
- Filter pending/reviewed/invalid annotations.
- Filter by class, confidence, model, and job.
- Show project/dataset statistics.
- Search jobs and inspect logs.
- Query export history.
- Compare model-generated and human-edited annotations.

### Ingestion and artifact workload
- Large dataset manifests.
- Batch annotation writes.
- Mask/polygon or RLE metadata.
- Keypoint arrays.
- Model outputs and confidence values.
- Job logs and result manifests.
- Model weights and temporary inference artifacts.
- Export packages.

### Scale requirements
Estimate or measure:
- Images and frames per project.
- Average image dimensions and file sizes.
- Number of annotations per image.
- Mask representation sizes.
- Keypoints per object.
- Number of classes.
- Job frequency and batch size.
- Concurrent workers.
- Local disk and memory constraints.
- Expected project growth.
- Export and retention requirements.

Do not fabricate numbers. Use representative scenarios marked **ESTIMATE** if required for capacity planning.

## 5. PHASE 3 — DATA MODEL DESIGN

Design a normalized relational schema where appropriate. Consider the following logical entities, but include only those supported by actual requirements:

- `projects`
- `datasets`
- `dataset_items` or `images`
- `video_sources` and `video_frames` if video support exists
- `categories`
- `keypoint_schemas`
- `keypoint_definitions`
- `skeleton_edges`
- `annotations`
- `annotation_geometries` or typed geometry payloads
- `models`
- `model_versions` / `checkpoints`
- `inference_profiles`
- `inference_jobs`
- `job_items` or per-image job results
- `job_artifacts`
- `review_events`
- `validation_reports`
- `export_jobs`
- `audit_events`

Do not create all these tables automatically. Determine whether a separate table, typed column, JSON field, or file reference best matches the actual access pattern.

For each entity document:
- Table/collection name.
- Business purpose.
- Primary key.
- Foreign keys.
- Unique constraints.
- Not-null constraints.
- Data types.
- Defaults.
- Relationships and cardinality.
- Indexes.
- Lifecycle and deletion rules.
- Audit requirements.

### Annotation data modeling
Define a canonical representation that can express:
- BBox: coordinate format, origin, normalization, image dimensions, class ID, confidence.
- SEG: instance/semantic mode, polygon or RLE encoding, mask dimensions, class association, optional confidence.
- Pose: keypoint schema ID, keypoint ordering, x/y coordinates, visibility state, confidence, skeleton edges, object/person association where applicable.

Keep task-specific fields explicit and validated. Avoid an opaque JSON-only model if common operations require relational querying. JSON/JSONB may be used for flexible model metadata or variable payloads when justified, with schema validation.

### Provenance
Preserve:
- Origin: model-generated, user-created, imported, or transformed.
- Model/checkpoint/version.
- Inference job and configuration.
- Creation/edit timestamps.
- Reviewer identity or local actor where relevant.
- Review status.
- Source annotation linkage when refined or edited.

Do not overwrite a model prediction when the user edits it if preserving prediction history is required. Define versioning or event-history behavior.

### File and artifact references
Store large binary data in filesystem/object storage where appropriate. The database should hold stable references and metadata, not necessarily the bytes themselves.

Define:
- Relative versus absolute paths.
- Project portability.
- File checksums.
- MIME/type and size.
- Atomic write strategy.
- Missing-file detection.
- Relocation/relink behavior.
- Temporary artifact cleanup.
- Backup/restore relationship between database and files.

## 6. PHASE 4 — NORMALIZATION AND DATA INTEGRITY

Apply suitable normalization, typically up to 3NF for transactional data where appropriate.

Evaluate:
- Functional dependencies.
- Duplicate data.
- Update/insert/delete anomalies.
- Class/category references.
- Dataset-item uniqueness.
- Annotation-to-image relationships.
- Job-to-result relationships.
- Keypoint schema consistency.
- Export snapshot consistency.

Enforce critical rules at the database layer where possible:
- Foreign keys.
- Unique project/dataset item identifiers.
- Valid status values.
- Required fields.
- Numeric and coordinate constraints where feasible.
- Referential integrity.
- Safe delete behavior.

Do not rely only on frontend validation. Avoid uncontrolled cascade deletes of projects, datasets, annotations, and job artifacts. Define explicit deletion and archival policy.

## 7. PHASE 5 — INDEXING AND QUERY OPTIMIZATION

Design indexes based on actual query patterns.

Consider:
- Primary and unique indexes.
- Composite indexes for project/dataset/status queries.
- Indexes for image ordering and job state.
- Partial indexes for pending/review states.
- JSON indexes only when JSON querying is justified.
- Full-text search only if textual search is required.

For each index explain:
- Query accelerated.
- Column order.
- Storage/write cost.
- Evidence or expected access pattern.

Review:
- N+1 queries.
- Full scans.
- Expensive annotation joins.
- Large dashboard aggregations.
- Repeated file metadata lookups.
- Unbounded job/log queries.
- Lock contention during batch writes.

Use `EXPLAIN`/`EXPLAIN ANALYZE` when supported and safe. Never run destructive or expensive analysis against production without approval.

## 8. PHASE 6 — LARGE DATASET AND ANNOTATION HANDLING

### Dataset indexing
- Use incremental scanning and stable IDs.
- Avoid storing duplicate copies of source images unless required.
- Store dimensions, format, size, checksum, and source path.
- Detect missing or changed files.
- Do not load every image into memory at once.

### Annotation retrieval
- Fetch annotations for the visible image/frame.
- Use pagination or bounded queries for annotation lists.
- Support filters by task, class, review status, model, and confidence.
- Keep geometry payloads compact and versioned.
- Avoid excessive database round trips while navigating images.

### Batch inference writes
- Use bounded batch transactions.
- Persist job/item results safely.
- Define partial batch failure semantics.
- Support idempotent re-import of remote results.
- Avoid holding transactions open during inference or network calls.

### Large masks and artifacts
- Evaluate polygon versus RLE versus file-backed mask storage.
- Store large binary masks as files when appropriate.
- Include dimensions, encoding, checksum, and relative path.
- Define cleanup for temporary masks and remote downloads.

### Aggregation
- Calculate counts and summaries from database queries.
- Cache or precompute only when measured performance requires it.
- Do not load all annotations into the desktop UI to calculate simple statistics.

## 9. PHASE 7 — PERFORMANCE AND SCALABILITY

### Stage 1 — Local desktop baseline
- One embedded database where suitable (e.g. SQLite).
- Project-scoped storage layout.
- Efficient indexes.
- Bounded transactions.
- Lazy image/annotation loading.
- Incremental dataset indexing.
- Background workers for heavy tasks.
- Atomic file writes.
- Automated project backup/export.

### Stage 2 — Larger local projects
Evaluate:
- WAL mode or equivalent concurrency tuning where supported.
- Batch inserts and updates.
- File-backed mask storage.
- Partitioned project artifacts.
- Search/index optimization.
- Optional analytical engine for reporting.
- Recovery and integrity checks.

### Stage 3 — Multi-user or centralized deployment
Only if product requirements justify it, evaluate:
- PostgreSQL as shared source of truth.
- Authentication and authorization.
- Concurrent annotation locking/versioning.
- Centralized artifact/object storage.
- Background job service.
- Queue/worker infrastructure.
- Read replicas or analytical storage at demonstrated scale.

Do not implement future-stage infrastructure prematurely. Define measurable triggers for each upgrade.

## 10. PHASE 8 — TRANSACTION MANAGEMENT AND CONCURRENCY

Define:
- Transaction boundaries for annotation edits and job result imports.
- Isolation and locking needs.
- Concurrent modification behavior.
- Optimistic version checks where useful.
- Unique conflict handling.
- Idempotency keys for remote result ingestion.
- Retry behavior.
- Deadlock or database-busy handling.
- Atomic relationship between database rows and file artifacts.

Do not keep transactions open during model inference or remote Kaggle execution. Use explicit job states and short transactions.

## 11. PHASE 9 — SECURITY AND PRIVACY

Implement:
- Least-privilege database access.
- Safe parameterized queries.
- Secure local configuration.
- No credentials in source control.
- Kaggle tokens in secure OS/user configuration where possible.
- No token or sensitive dataset paths in logs where avoidable.
- Explicit consent before remote data upload.
- Data minimization for remote packages.
- Defined remote artifact cleanup and local retention.
- Safe file path validation.
- Backup protection.
- User-controlled project deletion and export.

Do not expose database connection strings, Kaggle tokens, or private dataset metadata in logs or UI diagnostics.

## 12. PHASE 10 — MIGRATIONS AND VERSION CONTROL

Use the migration system compatible with the actual project stack.

Examples:
- Alembic for SQLAlchemy.
- Django migrations for Django.
- Prisma Migrate for Prisma.
- TypeORM migrations for TypeORM.
- Versioned SQL migrations for a lightweight SQLite application where appropriate.

Every schema change must be version-controlled, reproducible, ordered, reviewable, and tested.

For desktop projects:
- Define database schema version.
- Provide safe upgrade on project open.
- Back up before risky migrations.
- Handle interrupted migration.
- Maintain compatibility or provide explicit migration errors.
- Never silently discard user annotations.

Document rollback and project backup requirements. Do not casually drop columns/tables or rewrite user data.

## 13. PHASE 11 — LOCAL DEVELOPMENT AND PROJECT STORAGE

Provide a simple local setup that matches the actual application.

For a local-first product, do not require Docker/PostgreSQL if an embedded database and filesystem storage meet requirements.

Define a project layout, for example (adapt to actual implementation):

```text
<project-root>/
  project-config.json
  database/
    project.db
    migrations/
  datasets/
    manifest.json
    source-references/
  annotations/
    canonical/
    masks/
  models/
    model-metadata/
  jobs/
    job-records/
    logs/
    artifacts/
  exports/
  backups/
```

This is a proposed logical layout, not a requirement to duplicate source image files. Prefer references to original datasets when safe and portable.

Provide:
- `.env.example` only for actual environment configuration.
- Database initialization and migration instructions.
- Sample project metadata.
- Seed classes only for demo/test projects.
- Health/integrity checks.
- Backup and restore instructions.
- Database inspection instructions using suitable tools.

Never put real credentials in committed configuration files.

## 14. PHASE 12 — DEPLOYMENT AND PROJECT PORTABILITY

For desktop distribution, document:
- Database location and permissions.
- Project folder portability.
- Relative path strategy.
- Model weight storage and download policy.
- File locking and concurrent process behavior.
- Upgrade/migration process.
- Backup before upgrade.
- Export/restore project bundle.
- OS-specific path handling.
- Disk capacity and cleanup.

If a future server mode is proposed, clearly separate it from current desktop deployment.

## 15. PHASE 13 — BACKUP, RECOVERY, AND MONITORING

Define:
- Project backup scope (database plus referenced artifacts/configuration).
- Backup frequency or user-triggered backup behavior.
- Retention.
- Integrity/checksum verification.
- Restore procedure.
- Recovery after crash during annotation write.
- Recovery after interrupted inference.
- Orphan artifact cleanup.
- Missing source-file detection.
- Database health/integrity checks.

Do not claim backups work until restore has been tested. Provide a recovery runbook.

## 16. PHASE 14 — REQUIRED DATABASE DOCUMENTATION

Preserve existing documentation structure. Create or update relevant files:

```text
docs/03-du-lieu/
  database/
    DATABASE_AUDIT.md
    DATABASE_SELECTION.md
    DATABASE_ARCHITECTURE.md
    DATA_MODEL.md
    ERD.md
    TABLE_DICTIONARY.md
    ANNOTATION_SCHEMA.md
    PROVENANCE_AND_REVIEW.md
    FILE_STORAGE_STRATEGY.md
    INDEXING_STRATEGY.md
    QUERY_OPTIMIZATION.md
    MIGRATION_AND_VERSIONING.md
    PROJECT_PORTABILITY.md
    BACKUP_AND_RECOVERY.md
    SCALABILITY_ROADMAP.md
```

### TABLE_DICTIONARY.md
For each table/entity document:
- Name and business meaning.
- Column/field name.
- Data type.
- Nullability/default.
- Key constraints.
- Description.
- Indexes.

### ANNOTATION_SCHEMA.md
Document BBox, SEG, and Pose geometry, coordinate conventions, class references, confidence, provenance, review state, and schema version.

### SCALABILITY_ROADMAP.md
Describe when a local database should remain local and what measurable workload requirements would justify PostgreSQL or additional infrastructure.

## 17. PHASE 15 — TESTING AND VALIDATION

### Schema tests
- Required fields.
- Primary/foreign keys.
- Unique/check constraints.
- Valid state values.
- Schema version.

### Annotation integrity tests
- BBox coordinate bounds and ordering.
- Polygon/RLE validity.
- Mask dimensions and file references.
- Keypoint schema and coordinate validity.
- Category references.
- Provenance and review status.
- Image/frame references.

### Migration tests
- Fresh project initialization.
- Upgrade from previous schema versions.
- Interrupted migration recovery.
- Preservation of existing annotations.

### Data integrity tests
- Duplicate import prevention.
- Referential integrity.
- Transaction rollback.
- Idempotent remote result import.
- File/database consistency.
- Project relocation and relinking.

### Performance tests
- Dataset indexing.
- Image navigation and annotation retrieval.
- Batch inference result persistence.
- Review filters.
- Dashboard aggregation.
- Export generation.
- Concurrent worker writes where supported.

Use representative synthetic datasets. Document dataset size, hardware, concurrency, query parameters, execution time, throughput, and memory/disk use. Do not extrapolate a small benchmark into guaranteed capacity.

## 18. ACCEPTANCE CRITERIA

- [ ] Database choice is justified for desktop-first requirements.
- [ ] Existing business entities and data are audited.
- [ ] Canonical annotation schema supports BBox, SEG, and Pose.
- [ ] Provenance and review history are preserved.
- [ ] Project/dataset/image/job relationships are correct.
- [ ] Large binary artifacts use suitable file storage.
- [ ] Indexes match real query patterns.
- [ ] Dataset browsing and annotation retrieval are bounded and efficient.
- [ ] Batch writes and remote result imports are safe/idempotent.
- [ ] Migrations are version-controlled and tested.
- [ ] Project portability and recovery are documented.
- [ ] Kaggle credentials and remote data handling are secure.
- [ ] Backup and restore are tested.
- [ ] Performance claims have evidence.
- [ ] Scaling recommendations have measurable triggers.

## 19. FINAL HANDOVER

Provide:
1. Database audit and selection report.
2. Database architecture.
3. ERD and Mermaid source.
4. Table/entity dictionary.
5. Canonical BBox/SEG/Pose annotation schema.
6. Provenance and review data model.
7. File/artifact storage strategy.
8. Indexing and query optimization report.
9. Migration files and upgrade plan.
10. Local project storage layout.
11. Backup and recovery runbook.
12. Performance test report.
13. Security checklist.
14. Scalability roadmap.
15. Known limitations and upgrade triggers.

Clearly distinguish implemented changes from recommendations.

**FINAL PRINCIPLE: DESIGN THE DATA LAYER FOR REAL COMPUTER-VISION DATASETS AND TRUSTWORTHY ANNOTATIONS. KEEP PROJECT METADATA, ANNOTATION RECORDS, AND LARGE FILE ARTIFACTS IN THE RIGHT PLACES, AND NEVER SACRIFICE USER DATA FOR A SCHEMA CHANGE.**
