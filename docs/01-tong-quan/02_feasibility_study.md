# 02 — Feasibility Study

> Đánh giá tính khả thi về kỹ thuật, phần cứng, quy mô dữ liệu, model, đóng gói, Kaggle remote, chi phí và rủi ro, kèm kết luận cho MVP.
> Trạng thái: **Draft** · Cập nhật: 2026-11-24 · Dữ kiện lấy từ [03](03_research_and_references.md)

## 1. Kết luận nhanh

| Hạng mục | Đánh giá | Điều kiện |
|---|:-:|---|
| Desktop app Python + PySide6 có canvas annotation | 🟢 Khả thi | Spike hiệu năng `QGraphicsView` ở đầu M3 |
| BBox/SEG/POSE thủ công + review + export YOLO/COCO | 🟢 Khả thi | Công nghệ phổ biến, rủi ro thấp |
| Auto-label bằng YOLO (detect/segment/pose) | 🟢 Kỹ thuật / 🔴 **License** | Bị chặn bởi ADR-010 (AGPL-3.0) |
| SAM 2 refinement và interactive | 🟢 Khả thi | Dùng qua `transformers` để chạy được Windows native; trên CPU sẽ chậm (cần đo) |
| SAM 3 concept segmentation | 🟡 Có điều kiện | Cần GPU CUDA ≥ 12.6, weights gated, license tùy chỉnh → V1, ưu tiên chạy remote |
| DINOv2 similarity/dedup | 🟢 Khả thi | V1, license Apache 2.0 |
| DINOv3 | 🟡 Có điều kiện | Gated, license tùy chỉnh; head detect/segment cần backbone 7B → Future, chỉ dùng embedding |
| Hardware probe + compute router | 🟢 Khả thi | NVML + torch; profile phải được đo thực tế |
| Kaggle remote qua CLI | 🟡 Khả thi với giới hạn | Không hủy được kernel qua CLI; quota và thời gian phiên không đảm bảo; **rủi ro về điều khoản** |
| Đóng gói Windows | 🟡 Khả thi, phức tạp | Thin app + Runtime Manager (ADR-011) |

**Kết luận:** MVP **khả thi về kỹ thuật**. Có hai điểm cần quyết định của chủ dự án trước khi bắt đầu M5 và M8: (1) license YOLO, (2) có chấp nhận rủi ro điều khoản của Kaggle không.

## 2. Khả thi kỹ thuật

| Bài toán | Phân tích |
|---|---|
| UI không treo | Inference chạy ở worker process; I/O chạy trong QThreadPool; DB chỉ có một writer. Đây là các mẫu thiết kế đã được kiểm chứng rộng rãi trong ứng dụng Qt |
| Canvas | `QGraphicsView` đã được nhiều công cụ annotation desktop sử dụng; rủi ro nằm ở ảnh rất lớn (> 50 MP) và số annotation lớn → dùng preview, LOD, giới hạn số handle hiển thị |
| SAM interactive | Embedding ảnh tính một lần cho mỗi ảnh (nặng), các prompt sau đó nhẹ hơn; giữ model trong worker để tránh load lại. Độ trễ thực tế trên GPU/CPU của máy mục tiêu **chưa được đo** |
| Canonical schema và round-trip | Logic thuần, test được hoàn toàn |
| Chạy remote cho cùng kết quả | Dùng cùng một gói worker (ADR-006) |

## 3. Phần cứng

| Cấu hình | Có thể làm | Ghi chú |
|---|---|---|
| CPU-only, RAM 8–16 GB | Annotation thủ công, YOLO n/s (chậm), SAM 2 tiny/small interactive (chậm), Kaggle remote | Cảnh báo tốc độ; khuyến nghị chạy remote cho batch lớn |
| GPU NVIDIA cũ (Pascal) | Như trên, cộng GPU với torch 2.14 | Không chạy được runtime CUDA 13 |
| GPU Turing+ VRAM 4–6 GB | YOLO mọi cỡ với batch nhỏ, SAM 2 tiny→base+ | VRAM thực tế phải benchmark |
| GPU VRAM ≥ 8–12 GB | Thêm SAM 2 large, pipeline B với batch lớn hơn; SAM 3 có thể chạy được | **Chưa có số đo**; các con số VRAM này chỉ là giả định để phân nhóm, không phải yêu cầu |

Câu hỏi mở số 2 ([00 §8](00_project_overview.md)) sẽ quyết định profile mặc định.

## 4. Quy mô dataset (ESTIMATE, cần benchmark)

| Quy mô | Tác động |
|---|---|
| ≤ 10k ảnh | Không có vấn đề đặc biệt |
| 10k–50k (mục tiêu MVP) | Bắt buộc có dataset browser ảo hóa, thumbnail nền, index DB, ghi theo batch |
| 50k–200k | Kiến trúc không chặn, nhưng cần đo lại tốc độ quét, kích thước DB và thumbnail cache (dung lượng ổ đĩa) |
| > 200k | Ngoài phạm vi đã đánh giá |

Dung lượng thumbnail ước tính khoảng ~15–30 KB/ảnh (ESTIMATE với WebP 256 px). 50k ảnh tương đương khoảng ~1–1.5 GB, cần đo thực tế.

## 5. Kaggle remote

| Khía cạnh | Đánh giá |
|---|---|
| Kỹ thuật | Khả thi: `datasets create -r zip` → `kernels push -t` → `status` → `output` |
| Độ trễ | Phải chờ xếp hàng và khởi động kernel (**chưa đo**) → chỉ phù hợp với batch, không dùng cho interactive |
| Kiểm soát | Không hủy được kernel qua CLI → bắt buộc đặt `timeout` và chia chunk nhỏ |
| Quota | Khoảng 30 giờ GPU/tuần (UNVERIFIED), thay đổi được → không thể làm cam kết cho sản phẩm |
| Dữ liệu | Phải upload ảnh; bị giới hạn kích thước dataset/output (UNVERIFIED) |
| Pháp lý | ⚠️ Điều khoản có thể giới hạn mục đích sử dụng ([16 §7](../07-van-hanh/16_security_privacy_and_licenses.md)) |

**Đánh giá:** phù hợp làm tính năng "best-effort" cho người dùng cá nhân hoặc học tập (persona P3). **Không** nên coi là hạ tầng có SLA.

## 6. Đóng gói

Gói cài nhỏ, còn runtime được cài khi chạy lần đầu (ADR-011). Rủi ro gồm: lần đầu cần mạng, antivirus báo nhầm, chi phí ký số. Có phương án dự phòng là bundle sẵn torch-CPU.

## 7. Chi phí

| Hạng mục | Chi phí |
|---|---|
| Thư viện chính (PySide6, PyTorch, transformers, SAM 2, DINOv2) | Miễn phí (license mở) |
| Ultralytics YOLO trong sản phẩm closed-source | **Enterprise License, giá theo báo giá** (chưa biết) |
| Kaggle GPU | Miễn phí trong giới hạn quota (không đảm bảo) |
| Chứng chỉ ký số cho Windows | Có phí (tùy nhà cung cấp) |
| Phần cứng để test | Cần ít nhất một máy có GPU NVIDIA Turing+ và một máy không có GPU |
| Tư vấn pháp lý | Khuyến nghị trước khi phát hành thương mại |

## 8. Rủi ro chính

| ID | Rủi ro | Khả năng | Ảnh hưởng | Giảm thiểu |
|---|---|:-:|:-:|---|
| R-01 | License AGPL của YOLO xung đột với mô hình phân phối | Cao | Cao | ADR-010; adapter có thể thay thế |
| R-02 | Điều khoản hoặc quota của Kaggle thay đổi, hoặc không cho phép kiểu sử dụng này | Trung bình | Trung bình | Remote là tùy chọn; có provider interface; giữ local là mặc định |
| R-03 | Hiệu năng canvas với ảnh lớn hoặc nhiều annotation | Trung bình | Cao | Spike đầu M3 |
| R-04 | SAM 2 chạy trên CPU quá chậm cho interactive | Trung bình | Trung bình | Dùng checkpoint tiny; cảnh báo; khuyến nghị dùng GPU |
| R-05 | Ma trận driver/torch/CUDA gây lỗi cài đặt | Cao | Trung bình | Runtime Manager + diagnostics + troubleshooting |
| R-06 | Thư viện ML thay đổi API liên tục | Cao | Trung bình | Pin phiên bản, contract test cho adapter |
| R-07 | Mất dữ liệu annotation | Thấp | Rất cao | WAL, transaction, autosave, backup, kill test |
| R-08 | Phạm vi phình to | Trung bình | Cao | Phân tầng MVP/V1/Future; thay đổi phạm vi phải có ADR |

## 9. Khả thi của MVP

- **MVP-α** (BBox end-to-end): rủi ro thấp; chỉ phụ thuộc ADR-010. Nếu chọn phương án C thì phải làm thêm spike detector.
- **MVP đầy đủ:** khả thi với các điều kiện trên. Phần bất định lớn nhất là hiệu năng (cần đo) và các phụ thuộc bên ngoài (license, Kaggle).
