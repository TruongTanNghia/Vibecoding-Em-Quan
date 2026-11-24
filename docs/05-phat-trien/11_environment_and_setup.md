# 11 — Environment & Setup

> Hệ điều hành, phiên bản Python, ma trận dependency, tương thích CUDA, các bước cài đặt, smoke test, cùng cách thiết lập local và Kaggle CLI.
> Trạng thái: **PROPOSAL**. Phiên bản lấy theo PyPI và tài liệu chính thức ngày 2026-11-24. Các lệnh setup là **kế hoạch**, sẽ được xác minh khi bootstrap ở Bước 05.

## 1. Hệ điều hành mục tiêu

| OS | Mức hỗ trợ | Ghi chú |
|---|---|---|
| Windows 10/11 x64 | **Chính** | Đường dẫn Unicode và dài; high-DPI |
| Ubuntu 22.04+/Linux x64 | Best-effort (MVP), chính thức (V1) | Qt cần các thư viện xcb/wayland của hệ thống |
| macOS | Roadmap | Không có CUDA; cần đánh giá MPS |

## 2. Chọn phiên bản Python: **3.12** (ADR-008)

| Thành phần | Yêu cầu Python (đã xác minh) |
|---|---|
| PySide6 6.12 | ≥ 3.10, < 3.16 |
| PyTorch 2.15 | ≥ 3.11 |
| SAM 3 (repo Meta) | ≥ 3.12 |
| SAM 2 (repo Meta) | ≥ 3.10 |
| `ultralytics` 8.4.x | ≥ 3.8 |
| `kaggle` CLI 2.2.x | ≥ 3.11 |
| Nuitka 4.3 (nếu dùng để đóng gói) | ≤ 3.14 |
| PyInstaller 6.22 | ≥ 3.8, < 3.16 |

Vì vậy **3.12** đáp ứng được mọi thành phần, kể cả SAM 3. 3.13 cũng khả thi, nhưng ít được các thư viện ML kiểm thử hơn (cần đánh giá lại ở Bước 05).

## 3. Hai môi trường tách biệt (ADR-003, ADR-009)

| Môi trường | Chứa | Lý do tách |
|---|---|---|
| **App env** | PySide6, core, persistence, import/export, kaggle CLI (cài riêng dạng tool) | Nhẹ; không phụ thuộc phiên bản torch/CUDA |
| **Worker runtime env** | PyTorch (bản CPU hoặc CUDA phù hợp driver), `ultralytics`, `transformers`, OpenCV, worker package | Torch rất lớn và phụ thuộc driver; có thể có nhiều runtime khác nhau (ví dụ một bản riêng cho SAM 3) |

Worker runtime được tạo bằng **`uv`** (`uv venv --python 3.12` + `uv pip install`). Khi đóng gói, app chạy **Runtime Manager** để tạo runtime này lúc chạy lần đầu, sau khi người dùng đồng ý (xem [15](../07-van-hanh/15_packaging_release_and_operations.md)).

## 4. Ma trận dependency (PROPOSAL, sẽ pin chính xác trong lock file)

### 4.1 App env

| Package | Mục đích | Ràng buộc |
|---|---|---|
| `PySide6` | UI | 6.12.x (LGPL-3.0) |
| `numpy` | Geometry, RLE | |
| `Pillow` | Đọc ảnh, EXIF, thumbnail | |
| `pydantic` | Schema, config, validate manifest | v2 |
| `platformdirs` | Thư mục dữ liệu chuẩn của từng OS | |
| `psutil` | CPU/RAM/disk | |
| `nvidia-ml-py` | Đọc VRAM/driver mà không cần torch | 13.x, import bằng tên `pynvml` |
| `keyring` | Secret store (V1) | |
| `pycocotools` hoặc RLE tự viết | Encode/decode RLE kiểu COCO | Wheel Windows **UNVERIFIED** → nếu không có thì tự cài đặt RLE bằng numpy, có test đối chiếu |
| `kaggle` | Remote CLI | **≥ 2.2.5**, cài riêng bằng `uv tool install` |

### 4.2 Worker runtime

| Package | Ràng buộc |
|---|---|
| `torch`, `torchvision` | Chọn theo §5 |
| `ultralytics` | Pin chính xác (ví dụ `8.4.x`). Phát hành rất thường xuyên, nên không dùng `>=` |
| `transformers` | Pin; ≥ 4.56 nếu dùng DINOv3; phiên bản tối thiểu cho `Sam3Model` **UNVERIFIED** |
| `opencv-python-headless` | Mask → contour |
| `autolabel_worker` | Package worker của dự án (cùng version với app) |

**Optional extras** (gợi ý): `worker[yolo]`, `worker[sam2]`, `worker[sam3]`, `worker[dino]`. Chỉ cài phần người dùng cần.

## 5. Tương thích PyTorch – CUDA – driver (đã xác minh 2026-11-24)

| Máy người dùng | Runtime đề xuất |
|---|---|
| GPU NVIDIA **Turing (sm_75) trở lên** + driver **≥ 580** | `torch 2.15` + `cu132` hoặc `cu134` |
| GPU Pascal/Volta, **hoặc** driver 525–579 | `torch 2.14` + bản CUDA 12.x (tên variant chính xác **UNVERIFIED**) |
| Không có GPU NVIDIA | Bản CPU-only (`download.pytorch.org/whl/cpu`); có bản 2.15+cpu cho Windows hay không: **UNVERIFIED** |
| SAM 3 | Cần CUDA ≥ 12.6, PyTorch ≥ 2.7 → không chạy được trên CPU một cách thực tế |

- PyTorch 2.15 chỉ phát hành wheel `cu132` và `cu134` (đã bỏ cu126/cu130). Wheel CUDA 13 cần driver ≥ 580 và GPU từ Turing trở lên.
- Runtime Manager đọc driver và compute capability qua NVML, rồi chọn index URL phù hợp. Nếu không chắc chắn thì đề xuất bản CPU và để người dùng tự chọn.
- Wheel PyTorch đã kèm CUDA runtime, nên **không cần cài CUDA Toolkit**, chỉ cần driver NVIDIA.

## 6. Thiết lập môi trường phát triển (kế hoạch)

### 6.1 Windows PowerShell
```powershell
# Cài uv (theo hướng dẫn chính thức của uv), sau đó:
uv python install 3.12
cd <repo>
uv sync                                  # app env theo pyproject + uv.lock
# Worker runtime (ví dụ GPU Turing+, driver >= 580):
uv venv .runtime\worker --python 3.12
uv pip install --python .runtime\worker torch torchvision --index-url https://download.pytorch.org/whl/cu132
uv pip install --python .runtime\worker -e ".\worker[yolo,sam2]"
# Kaggle CLI (tùy chọn, cô lập):
uv tool install "kaggle>=2.2.5"
# Smoke test
uv run python -m autolabel_studio.diagnostics
uv run python -m autolabel_studio
```

### 6.2 Linux Bash
```bash
uv python install 3.12
uv sync
uv venv .runtime/worker --python 3.12
uv pip install --python .runtime/worker torch torchvision --index-url https://download.pytorch.org/whl/cpu
uv pip install --python .runtime/worker -e "./worker[yolo,sam2]"
uv run python -m autolabel_studio.diagnostics
```

> Các lệnh trên phụ thuộc vào cấu trúc repo ở [12](12_codebase_structure.md) và sẽ được xác minh trong M1. Không coi chúng là đã chạy thử.

## 7. Diagnostics / smoke test (`autolabel_studio.diagnostics`)

Kiểm tra và in báo cáo PASS/WARN/FAIL cho:
1. Phiên bản Python, OS, đường dẫn dữ liệu, quyền ghi.
2. Import PySide6 và tạo `QApplication` offscreen.
3. SQLite: phiên bản, có hỗ trợ WAL và JSON1 không.
4. Worker runtime: tìm thấy không, `hello` qua giao thức, phiên bản torch, `cuda.is_available()`, smoke test tensor.
5. NVML: driver, GPU, VRAM.
6. Model cache: dung lượng, các checkpoint đã có và checksum.
7. Kaggle CLI (nếu đã cấu hình): tìm thấy executable, phiên bản ≥ 2.2.5, xác thực OK (không in token).

## 8. Biến môi trường

| Biến | Mục đích | Ghi chú |
|---|---|---|
| `AUTOLABEL_HOME` | Ghi đè thư mục dữ liệu app | Tùy chọn |
| `AUTOLABEL_WORKER_PYTHON` | Đường dẫn Python của worker runtime | Dùng khi dev |
| `AUTOLABEL_LOG_LEVEL` | `INFO` / `DEBUG` | |
| `HF_HOME` | Do app đặt cho worker, trỏ vào model cache | Không cần người dùng tự đặt |
| `KAGGLE_API_TOKEN` | Token của Kaggle CLI | **Người dùng tự đặt**; app không ghi ra file và không log |

`.env.example` chỉ chứa tên biến, không bao giờ chứa giá trị secret.

## 9. Vấn đề thường gặp khi setup

Xem [17](../07-van-hanh/17_troubleshooting.md): driver quá cũ cho CUDA 13, OpenMP clash (`libiomp5md.dll`) trên Windows khi dùng ultralytics, thiếu thư viện xcb của Qt trên Linux, đường dẫn Unicode.
