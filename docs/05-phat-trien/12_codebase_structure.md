# 12 — Codebase Structure

> Cấu trúc thư mục, trách nhiệm module, ranh giới import, entry point, vị trí của config và dữ liệu.
> Trạng thái: **PROPOSAL** (greenfield). Chỉ tạo module khi có milestone thực sự cần đến nó (theo nguyên tắc của Bước 05).

## 1. Quyết định: uv workspace với 3 package (ADR-009)

| Package | Phụ thuộc | Lý do |
|---|---|---|
| `autolabel_core` | numpy, pydantic | Schema, geometry, validation, RLE. Thuần Python, **không có Qt, không có torch**. Dùng chung cho app, worker local và worker trên Kaggle |
| `autolabel_studio` | core, PySide6, … | Ứng dụng desktop |
| `autolabel_worker` | core, torch, ultralytics/transformers (extras) | Tiến trình inference. Chạy được ở local và trên Kaggle với **cùng một mã nguồn** (ADR-006) |

So với cấu trúc tham khảo `src/auto_label_studio/` trong prompt Bước 05, cấu trúc này tách worker ra để upload lên Kaggle mà không kéo theo PySide6, và để cô lập dependency của torch.

## 2. Cây thư mục

```
autolabel-studio/
├── pyproject.toml              # uv workspace root, cấu hình ruff/mypy/pytest/import-linter
├── uv.lock
├── README.md · LICENSE · THIRD_PARTY_NOTICES.md · CHANGELOG.md
├── .gitignore · .env.example
├── packages/
│   ├── core/src/autolabel_core/
│   │   ├── schema/            # canonical annotation (pydantic), JSON Schema export, version
│   │   ├── geometry/          # bbox, polygon, mask/RLE, keypoints, transform, EXIF
│   │   ├── validation/        # rule engine + rule V-*
│   │   ├── states/            # state machine: image workflow, job (guard functions)
│   │   ├── protocol/          # message của giao thức app ↔ worker, manifest, prediction batch
│   │   └── errors.py          # mã lỗi E-*
│   ├── app/src/autolabel_studio/
│   │   ├── __main__.py        # entry point: python -m autolabel_studio
│   │   ├── bootstrap.py       # tạo QApplication, DI container, logging, excepthook
│   │   ├── ui/                # chỉ chứa Qt
│   │   │   ├── main_window/ · screens/ · canvas/ · tools/ · widgets/ · dialogs/
│   │   │   ├── viewmodels/    # presenter / view model gắn với use case
│   │   │   └── theme/         # QSS, palette, token màu
│   │   ├── application/       # use case: create_project, import_dataset, run_autolabel, review, export...
│   │   ├── domain/            # project, dataset, class/keypoint schema service (không có Qt)
│   │   ├── persistence/       # SQLite repository, migration, autosave, backup
│   │   ├── inference/         # registry, orchestrator, job queue, worker client (subprocess)
│   │   ├── compute/
│   │   │   ├── hardware/      # probe phía app (NVML/psutil)
│   │   │   ├── router/        # pure function router + policy
│   │   │   └── remote/
│   │   │       ├── provider.py          # interface RemoteComputeProvider
│   │   │       ├── kaggle/              # cli wrapper, packager, poller, collector
│   │   │       └── mock/                # mock provider cho test và demo offline
│   │   ├── io/
│   │   │   ├── importers/     # yolo_detect, yolo_seg, yolo_pose, coco_*, (voc)
│   │   │   └── exporters/
│   │   ├── services/          # thumbnail, file store, diagnostics bundle
│   │   ├── config/            # settings (pydantic), paths (platformdirs)
│   │   ├── runtime_manager/   # tạo/cập nhật worker runtime bằng uv (đóng gói)
│   │   └── logging/           # structlog/logging config, redact filter
│   └── worker/src/autolabel_worker/
│       ├── __main__.py        # python -m autolabel_worker {serve|run|probe}
│       ├── server.py          # vòng lặp JSON-lines stdin/stdout
│       ├── runner.py          # chạy batch theo manifest (local + Kaggle)
│       ├── probe.py           # hardware/torch probe
│       ├── adapters/
│       │   ├── base.py        # Protocol: BatchPredictor, PromptableSegmenter, EmbeddingExtractor
│       │   ├── yolo/          # detect, segment, pose
│       │   ├── sam2/ · sam3/ · dino/
│       │   └── fake/          # adapter giả lập (deterministic) cho test
│       └── kaggle_entry/runner_template.py
├── registry/models/*.yaml      # built-in model descriptor (pin checksum)
├── configs/                    # cấu hình mặc định, preset tham số
├── assets/                     # icon, banner (Bước 07)
├── scripts/                    # dev scripts: lint, gen_fixtures, benchmark
├── tests/
│   ├── unit/ · integration/ · contract/ · ui/ · e2e/ · perf/
│   └── fixtures/               # ảnh nhỏ tổng hợp, dataset YOLO/COCO mẫu
├── packaging/                  # PyInstaller spec, installer script
└── docs/
```

## 3. Trách nhiệm và luật import

| Module | Được import | **Không** được import |
|---|---|---|
| `autolabel_core` | stdlib, numpy, pydantic | PySide6, torch, ultralytics, transformers, `autolabel_studio`, `autolabel_worker` |
| `studio.domain`, `studio.application` | core, persistence (qua interface), inference, compute, io | `studio.ui`, PySide6 |
| `studio.ui` | application, viewmodels, PySide6 | persistence, compute.remote, worker (truy cập trực tiếp) |
| `studio.persistence` | core, sqlite3 | ui |
| `studio.compute.router` | core | mọi I/O (đây là pure function) |
| `studio.inference` | core, compute | torch (torch chỉ có ở worker) |
| `autolabel_worker` | core, torch, thư viện model | `autolabel_studio`, PySide6 |

Các luật này được thực thi tự động bằng **import-linter** (contract `forbidden` và `layers`) trong CI (NFR-MAINT-02).

**Ngoại lệ có kiểm soát:** Qt signal được dùng để đưa event lên UI, thông qua một lớp adapter `ui/bridge.py`. Application layer chỉ phát callback/event thuần Python.

## 4. Entry point

| Lệnh | Mục đích |
|---|---|
| `python -m autolabel_studio` | Chạy app |
| `python -m autolabel_studio.diagnostics` | Smoke test môi trường ([11 §7](11_environment_and_setup.md)) |
| `python -m autolabel_worker serve` | Worker ở chế độ JSON-lines (app tự spawn) |
| `python -m autolabel_worker run --manifest m.json --out dir` | Chạy batch không cần app (dùng cho Kaggle và test) |
| `python -m autolabel_worker probe` | In báo cáo phần cứng dạng JSON |

## 5. Vị trí config và dữ liệu (dùng `platformdirs`)

| Loại | Windows | Linux |
|---|---|---|
| Settings của app | `%APPDATA%\AutoLabelStudio\settings.json` | `~/.config/autolabel-studio/` |
| Dữ liệu app (model cache, runtime, danh sách project gần đây) | `%LOCALAPPDATA%\AutoLabelStudio\` | `~/.local/share/autolabel-studio/` |
| Log | `%LOCALAPPDATA%\AutoLabelStudio\logs\` | `~/.local/state/autolabel-studio/logs/` |
| Project | Thư mục workspace do người dùng chọn | như Windows |
| Temp của job | `<project>/jobs/<job_id>/` | như Windows |

Repository **không bao giờ** chứa dataset, model weights, cache hay log (xem `.gitignore` ở [16](../07-van-hanh/16_security_privacy_and_licenses.md)).

## 6. Quy ước code

- Python 3.12, type hint đầy đủ; `mypy --strict` cho `autolabel_core`, mức nới lỏng hơn cho ui.
- `ruff` (lint + format). Docstring cho API công khai của core và adapter.
- Tên module/hàm/biến tiếng Anh; chuỗi hiển thị trên UI tiếng Việt, gom lại để có thể dịch (`QCoreApplication.translate` / `.ts`).
- Comment tham chiếu quy tắc nghiệp vụ khi cần: `# BR-02: không ghi đè annotation human-verified`.
- ID dùng UUIDv7 (sắp xếp được theo thời gian), thời gian lưu UTC ISO-8601.
