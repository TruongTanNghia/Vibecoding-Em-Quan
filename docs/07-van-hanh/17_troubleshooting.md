# 17 — Troubleshooting

> Hướng dẫn xử lý sự cố theo triệu chứng: dependency, CUDA, VRAM, đọc ảnh, annotation hỏng, Kaggle, truyền dữ liệu, đóng gói.
> Trạng thái: **Draft**. Đây là danh sách dự kiến; sẽ được bổ sung bằng lỗi thực tế gặp trong quá trình implement. Mã lỗi theo [04 §11](../02-kien-truc/04_system_architecture.md).

Bước đầu tiên cho mọi sự cố: **Tools → Diagnostics** (hoặc `python -m autolabel_studio.diagnostics`), sau đó xem log ở `%LOCALAPPDATA%\AutoLabelStudio\logs\`.

## 1. Môi trường và dependency

| Triệu chứng | Nguyên nhân khả dĩ | Cách xử lý |
|---|---|---|
| App không khởi động, báo lỗi plugin `qt.qpa` (Linux) | Thiếu thư viện xcb/wayland của hệ thống | Cài các gói `libxcb-*` theo tài liệu Qt; thử `QT_QPA_PLATFORM=wayland` |
| `OMP: Error #15 ... libiomp5md.dll already initialized` (Windows) | Hai bản OpenMP bị nạp cùng lúc (đã có báo cáo khi dùng ultralytics + torch) | Chạy worker trong runtime riêng (thiết kế mặc định); không dùng `KMP_DUPLICATE_LIB_OK` trên bản phát hành vì có thể sai kết quả |
| Worker báo `incompatible_runtime` | Runtime thiếu extra (`sam2`, `yolo`…) hoặc sai phiên bản | Settings → Runtimes → Sửa / Cài extra |
| `ImportError` sau khi cập nhật app | Phiên bản giao thức của app và runtime không khớp | Runtime Manager → Cập nhật runtime |

## 2. CUDA và GPU

| Triệu chứng | Nguyên nhân | Cách xử lý |
|---|---|---|
| `torch.cuda.is_available() == False` dù có GPU NVIDIA | Đang dùng bản torch CPU, hoặc driver quá cũ cho bản CUDA của wheel | Hardware report: so sánh driver với yêu cầu. torch 2.15 (cu13x) cần driver ≥ 580 và GPU Turing+; máy cũ dùng torch 2.14 (cu12x) |
| Smoke test GPU fail dù `is_available()` vẫn True | Driver hỏng, GPU đang bị tiến trình khác chiếm, TDR timeout trên Windows | Khởi động lại; cập nhật driver; đóng ứng dụng khác đang dùng GPU |
| GPU Pascal (GTX 10xx) không chạy với runtime mới | CUDA 13 bỏ hỗ trợ kiến trúc cũ | Chọn runtime torch 2.14 |
| Không có tùy chọn SAM 3 trên máy | SAM 3 cần CUDA ≥ 12.6 và GPU đủ mạnh | Chạy SAM 3 qua Kaggle (V1) |

## 3. Bộ nhớ (VRAM/RAM)

| Triệu chứng | Cách xử lý |
|---|---|
| `E-CMP-003` OOM ngay cả với batch 1 | Giảm `imgsz`, dùng checkpoint nhỏ hơn (n/s, SAM 2 tiny), bật fp16, đóng phiên SAM interactive, chạy CPU hoặc Kaggle |
| Hết RAM khi import hoặc export dataset lớn | Kiểm tra có đang bật checksum và thumbnail song song không; giảm số thread nền trong Settings |
| VRAM không được giải phóng sau job | Worker unload theo idle timeout; có thể bấm Jobs → Giải phóng GPU (kết thúc worker) |

## 4. Đọc ảnh và dataset

| Triệu chứng | Nguyên nhân | Cách xử lý |
|---|---|---|
| `E-DS-004` nhiều ảnh bị "missing" | Ổ ngoài bị rút, thư mục bị đổi tên hoặc di chuyển | Dataset → Relink root |
| Box lệch hoặc xoay 90° so với ảnh | EXIF orientation không được xử lý thống nhất (thường gặp với annotation import từ công cụ khác) | Kiểm tra cột `exif_orientation`; khi import có tùy chọn "tọa độ theo ảnh thô (raw)" hay theo ảnh đã xoay |
| Ảnh 16-bit hoặc TIFF hiển thị tối | Ảnh đã được chuẩn hóa về 8-bit để hiển thị | Dùng Display → Auto contrast (V1); dữ liệu gốc không bị thay đổi |
| Tên file tiếng Việt gây lỗi | Thư viện bên ngoài không hỗ trợ Unicode path | Báo lỗi kèm log; app đọc ảnh qua Pillow bằng đường dẫn Unicode (có test) |

## 5. Annotation và database

| Triệu chứng | Cách xử lý |
|---|---|
| Mở project báo `integrity_check failed` | Khôi phục từ `backups/` (đã có snapshot trước migration và khi đóng project) |
| Migration thất bại | App tự khôi phục snapshot trước migration, project vẫn ở phiên bản cũ; gửi log kèm báo lỗi |
| Annotation mất sau crash | Kiểm tra khoảng debounce autosave; mở Diagnostics → Recovery để xem các thay đổi chưa flush (nếu có journal) |
| Export bị chặn bởi preflight | Mở S08, lọc theo mã luật V-*; sửa hoặc chọn hành động tự động có ghi nhận |

## 6. Kaggle remote

| Triệu chứng | Nguyên nhân | Cách xử lý |
|---|---|---|
| `E-RMT-001` không xác thực được | Chưa cấu hình token hoặc token đã hết hạn/bị thu hồi | Tạo token mới trên Kaggle Settings; đặt `KAGGLE_API_TOKEN` hoặc file token chuẩn; bấm Kiểm tra lại |
| Kernel lỗi ngay khi bắt đầu, log báo không có GPU | Tài khoản chưa xác minh số điện thoại, hết quota tuần, hoặc accelerator không khả dụng | Kiểm tra tài khoản; đổi `machine_shape`; chạy local; thử lại sau khi quota reset |
| Job ở `SUBMITTED` (QUEUED) rất lâu | Hàng đợi GPU của Kaggle | Chờ, hoặc Cancel (lưu ý kernel có thể vẫn chạy về sau) |
| Bấm Cancel nhưng quota vẫn bị trừ | CLI không có lệnh dừng kernel; kernel chạy cho tới khi hết `timeout` | Giảm kích thước chunk/timeout; đây là giới hạn đã biết ([09 §4](../02-kien-truc/09_kaggle_remote_worker.md)) |
| `E-RMT-007` sai checksum hoặc schema | Tải về không trọn vẹn, output thiếu, hoặc phiên bản worker không khớp | Tải lại output; kiểm tra `env.json` và `worker.log` trong thư mục job |
| Upload thiếu file con | Đã dùng dir-mode mặc định `skip` | App luôn dùng `-r zip`; nếu tự chạy CLI thủ công thì phải thêm cờ này |
| `EXPIRED` | Kernel vượt quá giới hạn phiên hoặc timeout | Resume: chỉ các chunk chưa xong được gửi lại |

## 7. Đóng gói và phân phối

| Triệu chứng | Cách xử lý |
|---|---|
| Windows SmartScreen chặn | Bản chưa được ký số; chọn "More info → Run anyway" (bản nội bộ), hoặc ký số cho bản phát hành |
| Antivirus cách ly file `.exe` | False positive thường gặp với PyInstaller; dùng one-folder, ký số, báo cáo false positive với nhà cung cấp antivirus |
| Runtime Manager tải torch thất bại | Mạng hoặc proxy; cho phép chọn thư mục chứa wheel offline (V1) |
