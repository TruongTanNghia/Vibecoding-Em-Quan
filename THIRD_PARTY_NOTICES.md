# Third-Party Notices

> Cập nhật: 2026-11-24

## Hiện trạng

Repository **chưa chứa mã nguồn, thư viện, model weights hay dataset của bên thứ ba**. Nội dung hiện có (tài liệu, SVG minh họa, cấu hình Claude Code) đều do tác giả tạo ra.

## Thành phần dự kiến sử dụng

Danh sách dưới đây là **kế hoạch**, dùng để đánh giá license trước khi tích hợp. Khi một thành phần thực sự được thêm vào, file này sẽ được cập nhật kèm phiên bản và toàn văn license notice bắt buộc.

| Thành phần | License (xác minh 2026-11-24) | Ghi chú |
|---|---|---|
| PySide6 / Qt | LGPL-3.0-only / GPL (có bản thương mại) | Liên kết động; phải đính kèm notice |
| PyTorch | BSD-style | |
| Hugging Face `transformers` | Apache-2.0 | |
| Ultralytics `ultralytics` + weights YOLO | **AGPL-3.0** / Enterprise License | ⚠️ Chưa có quyết định tích hợp (ADR-010) |
| SAM 2 / 2.1 (Meta) | Apache-2.0 | |
| SAM 3 / 3.1 (Meta) | SAM License (tùy chỉnh) | Gated; V1, có điều kiện |
| DINOv2 (Meta) | Apache-2.0 | Không dùng Cell-DINO hay XRay-DINO (non-commercial) |
| DINOv3 (Meta) | DINOv3 License (tùy chỉnh) | Gated; Future |
| `kaggle` CLI | Cần xác minh | Công cụ tùy chọn, người dùng tự cài |

Người dùng tự chịu trách nhiệm xác minh quyền sử dụng các model checkpoint và dữ liệu của mình. Chi tiết: [docs/07-van-hanh/16_security_privacy_and_licenses.md](docs/07-van-hanh/16_security_privacy_and_licenses.md).

Badge trong README dùng dịch vụ [shields.io](https://shields.io).
