# 03 — Research & References

> Kết quả xác minh công nghệ và model từ nguồn chính thức, kèm trạng thái kiểm chứng.
> Ngày xác minh: **2026-11-24**. Thông tin thay đổi rất nhanh (Ultralytics phát hành vài ngày một lần), nên **phải xác minh lại khi implement** và pin phiên bản chính xác.
> Nhãn: **CONFIRMED** (đã đọc nguồn chính thức) · **UNVERIFIED** (chỉ có nguồn phụ hoặc không truy cập được). Ghi chú về license **không phải tư vấn pháp lý**.

## 1. Model

### 1.1 Ultralytics YOLO
| Nội dung | Trạng thái |
|---|---|
| **YOLO26** là họ mới nhất được khuyến nghị (phát hành 01/2026; có các cỡ n/s/m/l/x; bỏ DFL; có chế độ NMS-free tùy chọn `nms=False`, mặc định vẫn dùng NMS). YOLO11 là lựa chọn ổn định. YOLO27 mới ở mức "coming soon" | CONFIRMED |
| Package `ultralytics` 8.4.218 (23/11/2026), phát hành vài lần mỗi tuần | CONFIRMED |
| Task: YOLO26 hỗ trợ detect, segment, semantic segmentation, depth, classify, pose, OBB; YOLO11/v8 hỗ trợ detect, segment, classify, pose, OBB | CONFIRMED |
| Yêu cầu: Python ≥ 3.8, PyTorch ≥ 1.8; trên Windows loại trừ torch 2.4.0 | CONFIRMED |
| License: **AGPL-3.0**; Ultralytics coi model đã train/fine-tune cũng là AGPL; sản phẩm closed-source cần **Enterprise License** | CONFIRMED (cách diễn giải về weights còn gây tranh cãi, cần ý kiến pháp lý) |
| Wrapper cho SAM, SAM 2/2.1, SAM 3 (chỉ predict, không export SAM 2) | CONFIRMED |
| Format label detect/segment/pose, các trường `kpt_shape` và `flip_idx` trong `data.yaml` | CONFIRMED |
| Ý nghĩa giá trị visibility trong label YOLO pose (giả định theo COCO 0/1/2) | UNVERIFIED |
| Checkpoint pose pretrained của YOLO26 dùng schema COCO-17 | UNVERIFIED (suy ra từ ví dụ `flip_idx`) |
| Export: `onnx`, `engine` (TensorRT), `openvino`, `torchscript`…; `half`/`int8` được thay bằng `quantize` | CONFIRMED |

Nguồn: https://docs.ultralytics.com/models/ · https://docs.ultralytics.com/models/yolo26/ · https://docs.ultralytics.com/quickstart/ · https://docs.ultralytics.com/datasets/detect/ · https://docs.ultralytics.com/datasets/segment/ · https://docs.ultralytics.com/datasets/pose/ · https://docs.ultralytics.com/modes/export/ · https://www.ultralytics.com/license · https://pypi.org/project/ultralytics/

### 1.2 SAM 2 / SAM 2.1
| Nội dung | Trạng thái |
|---|---|
| License **Apache 2.0** cho code và checkpoint (cc_torch dùng BSD-3, font demo dùng OFL) | CONFIRMED |
| Checkpoint: hiera tiny 38.9M, small 46M, base+ 80.8M, large 224.4M tham số (SAM 2 ra ngày 2024-07-29, SAM 2.1 ngày 2024-09-29) | CONFIRMED |
| Python ≥ 3.10, PyTorch ≥ 2.5.1, torchvision ≥ 0.20.1; CUDA extension tùy chọn | CONFIRMED |
| **Windows native không được hỗ trợ chính thức**, repo khuyến nghị WSL | CONFIRMED |
| Prompt dạng point (+/−), box, mask; hỗ trợ ảnh và video | CONFIRMED |
| `transformers`: `Sam2Model`, `Sam2Processor`, pipeline mask-generation; hỗ trợ video | CONFIRMED (tên class video UNVERIFIED) |
| Đường export ONNX/TensorRT chính thức | UNVERIFIED (chỉ có công cụ cộng đồng) |

Nguồn: https://github.com/facebookresearch/sam2 · https://huggingface.co/docs/transformers/model_doc/sam2

### 1.3 SAM 3 / SAM 3.1
| Nội dung | Trạng thái |
|---|---|
| Detect, segment và track **mọi instance** của một khái niệm, dùng text (noun phrase) và/hoặc exemplar làm prompt; vẫn nhận point/box/mask | CONFIRMED |
| ~848M tham số | CONFIRMED |
| Weights **gated** trên Hugging Face (phải xin quyền, rồi `hf auth login`) | CONFIRMED |
| License **"SAM License"** tùy chỉnh: royalty-free, cho phép phân phối/sửa; cấm dùng cho quân sự/ITAR/vũ khí; không reverse-engineer; indemnify Meta; luật California | CONFIRMED (có được dùng thương mại không: chưa rõ, cần ý kiến pháp lý) |
| Python ≥ 3.12, PyTorch ≥ 2.7, GPU CUDA ≥ 12.6; hướng dẫn cài theo kiểu Linux/Conda | CONFIRMED |
| `transformers`: `Sam3Model`, `Sam3Processor`, `Sam3TrackerModel`, `Sam3VideoModel` | CONFIRMED (phiên bản tối thiểu UNVERIFIED) |
| SAM 3.1 "Object Multiplex" (27/03/2026), gated tại `facebook/sam3.1` | CONFIRMED |

Nguồn: https://github.com/facebookresearch/sam3 · https://huggingface.co/facebook/sam3 · https://huggingface.co/docs/transformers/model_doc/sam3 · https://ai.meta.com/blog/segment-anything-model-3/

### 1.4 DINOv2
| Nội dung | Trạng thái |
|---|---|
| License **Apache 2.0** cho code và weights (trừ Cell-DINO, XRay-DINO là non-commercial) | CONFIRMED |
| Biến thể ViT-S/B/L/g /14, có bản with registers | CONFIRMED |
| Load bằng `torch.hub` hoặc `transformers` (`Dinov2Model`, `Dinov2WithRegistersModel`) | CONFIRMED |
| Có head segmentation và depth; **không có head detection** | CONFIRMED |

Nguồn: https://github.com/facebookresearch/dinov2 · https://huggingface.co/docs/transformers/model_doc/dinov2

### 1.5 DINOv3
| Nội dung | Trạng thái |
|---|---|
| License **"DINOv3 License"** tùy chỉnh (cấu trúc giống SAM License; Meta có thể thay đổi điều khoản); blog của Meta nói là "commercial license" | CONFIRMED |
| Gated (form đăng ký + Hugging Face) | CONFIRMED |
| ViT-S/S+/B/L/H+ /16 (distilled), ViT-7B; ConvNeXt T/S/B/L | CONFIRMED |
| Head detect (COCO) và segment (ADE20K) chính thức chạy trên backbone **7B** | CONFIRMED |
| `transformers` ≥ 4.56; PyTorch ≥ 2.7.1 cho training/eval; chỉ được test trên Linux | CONFIRMED |

Nguồn: https://github.com/facebookresearch/dinov3 · https://ai.meta.com/blog/dinov3-self-supervised-vision-model/

## 2. Desktop, runtime và đóng gói

| Thành phần | Thông tin | Trạng thái |
|---|---|---|
| PySide6 | 6.12.0 (08/10/2026); license LGPL-3.0-only / GPL-2.0 / GPL-3.0, có bản thương mại; Python ≥ 3.10, < 3.16 | CONFIRMED — https://pypi.org/project/PySide6/ |
| PyTorch | 2.15.0 (28/10/2026); Python ≥ 3.11; **chỉ có wheel `cu132`/`cu134`**, cần driver ≥ 580 và GPU Turing (sm_75) trở lên; GPU Maxwell/Pascal/Volta hoặc driver CUDA 12.x nên ở lại 2.14; có wheel Windows | CONFIRMED — https://github.com/pytorch/pytorch/releases |
| `torch.cuda.mem_get_info` | Trả về VRAM trống và tổng (cudaMemGetInfo) | CONFIRMED |
| Wheel CPU 2.15 cho Windows | Có index CPU nhưng chưa xác nhận được có bản 2.15+cpu | UNVERIFIED |
| Driver NVIDIA | CUDA 13.x cần ≥ 580; CUDA 12.x cần ≥ 525 | CONFIRMED — NVIDIA CUDA release notes |
| `nvidia-ml-py` | 13.615.71, import bằng `pynvml` | CONFIRMED |
| PyInstaller | 6.22.3, Python < 3.16; one-file giải nén ra thư mục tạm mỗi lần chạy | CONFIRMED — https://pyinstaller.org |
| Kích thước bundle torch+CUDA, giới hạn của one-file | ~3 GB; giới hạn archive/exe | UNVERIFIED (nguồn cộng đồng) |
| Nuitka | 4.3.5, AGPLv3+ với runtime exception, **Python ≤ 3.14** | CONFIRMED |
| uv | 0.13.9; `uv python install`, `uv venv --python` | CONFIRMED |

## 3. Kaggle CLI và nền tảng

| Nội dung | Trạng thái |
|---|---|
| `kaggle` 2.2.5; Python ≥ 3.11; khuyến nghị cài bằng pipx/uv | CONFIRMED — https://pypi.org/project/kaggle/ |
| Xác thực: `kaggle auth login` (OAuth qua trình duyệt), `KAGGLE_API_TOKEN`, `~/.kaggle/access_token`, `kaggle.json` legacy / `KAGGLE_USERNAME`+`KAGGLE_KEY` | CONFIRMED — https://github.com/Kaggle/kaggle-api/blob/main/docs/README.md |
| 2.2.5 yêu cầu certifi ≥ 2026.10.19 (để tránh lộ token khi bị MITM) | CONFIRMED — CHANGELOG |
| `datasets create/version`: mặc định private; `-r/--dir-mode` mặc định `skip` | CONFIRMED — docs/datasets.md |
| `kernels push` chạy kernel; có `--accelerator`, `-t/--timeout`, `--no-run`; `output --file-pattern`; `status`; `delete`; `logs` (có trong CHANGELOG) | CONFIRMED — docs/kernels.md |
| **Không có lệnh dừng kernel đang chạy qua CLI** | CONFIRMED (dựa trên việc không có trong docs và CHANGELOG) |
| Trạng thái `KernelWorkerStatus`: QUEUED, RUNNING, COMPLETE, ERROR, CANCEL_REQUESTED, CANCEL_ACKNOWLEDGED, NEW_SCRIPT | CONFIRMED — kagglesdk |
| `kernel-metadata.json`: id, title, code_file, language, kernel_type, is_private, enable_gpu, enable_internet, **machine_shape**, dataset_sources, kernel_sources, model_sources, competition_sources | CONFIRMED — docs/kernels_metadata.md |
| Accelerator: NvidiaTeslaT4 (T4×2), NvidiaL4, NvidiaL4X1, NvidiaTeslaA100, NvidiaH100, NvidiaRtxPro6000, TPU…; P100 đã ngừng | CONFIRMED (loại GPU mà tài khoản free thực sự nhận được: UNVERIFIED) |
| Quota GPU ~30 giờ/tuần; phiên 12 giờ; output 20 GB; dataset ~200 GB; cần xác minh số điện thoại | UNVERIFIED (trang docs render bằng JS, chỉ có snippet/forum) |
| AUP cấm lạm dụng tài nguyên; Terms có thể giới hạn ở mục đích "personal, non-commercial" | UNVERIFIED — https://www.kaggle.com/aup |

## 4. Format annotation

| Nội dung | Trạng thái |
|---|---|
| COCO: các trường annotation, bbox `[x,y,w,h]` từ góc trên-trái, 0-indexed; `iscrowd` 0 = polygon, 1 = RLE; keypoints `[x,y,v]×k`; `v` ∈ {0,1,2}; `num_keypoints` đếm các điểm có `v>0` | CONFIRMED — cocodataset.org format-data |
| COCO `skeleton` dùng chỉ số 1-based | UNVERIFIED (kiến thức chung, phải kiểm tra trên file chuẩn) |
| Pascal VOC: `xmin ymin xmax ymax` pixel, thường được hiểu là 1-based | UNVERIFIED (trang devkit không truy cập được) → làm thành tùy chọn có thể cấu hình |

## 5. Hệ quả đối với thiết kế

1. **License của YOLO là quyết định chặn** (ADR-010).
2. SAM 2 dùng qua `transformers` để chạy native trên Windows (ADR-007).
3. SAM 3 chỉ khả thi trên GPU CUDA ≥ 12.6 → là ứng viên cho chạy remote; weights gated (V1).
4. DINO chỉ dùng cho embedding; không dựa vào head detect/segment của DINOv3, vì chúng yêu cầu backbone 7B.
5. Torch phải được chọn theo driver → cần Runtime Manager (ADR-011).
6. Python 3.12 (ADR-008).
7. Kaggle: không hủy được qua CLI → bắt buộc đặt timeout; xác thực bằng token; luôn dùng `-r zip`; có rủi ro về điều khoản.

## 6. Việc cần xác minh khi implement

- [ ] Ý nghĩa visibility trong YOLO pose (đọc source `ultralytics`).
- [ ] Schema keypoint của checkpoint pose YOLO26 (`kpt_shape` khi load).
- [ ] Phiên bản `transformers` tối thiểu cho `Sam3Model`; hiệu năng của `Sam2Model` so với repo gốc.
- [ ] Wheel torch CPU 2.15 cho Windows; tên variant CUDA 12.x của torch 2.14.
- [ ] Giới hạn của Kaggle (quota, thời gian phiên, output, dataset), lệnh xóa dataset qua CLI, cách cài secret qua CLI, định dạng output của `kernels status`.
- [ ] Wheel Windows của `pycocotools`.
- [ ] License của `kaggle` CLI.
- [ ] Ý kiến pháp lý: Ultralytics AGPL, SAM License, DINOv3 License, điều khoản Kaggle.
