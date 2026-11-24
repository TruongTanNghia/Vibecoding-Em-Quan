# 16 — Security, Privacy & Licenses

> Quyền riêng tư dữ liệu local, Kaggle credential, consent truyền dữ liệu, license của model và thư viện, quản lý secret, threat model.
> Trạng thái: **PROPOSAL** · Cập nhật: 2026-11-24 · ⚠️ Phần license là **phân loại kỹ thuật, không phải tư vấn pháp lý**. Trước khi phát hành phải có ý kiến pháp lý.

## 1. Quyền riêng tư

| Nguyên tắc | Hiện thực |
|---|---|
| Local-first | Dataset, annotation và project nằm trên máy người dùng; app không có telemetry |
| Không có kết nối mạng ngầm | Chỉ có 3 loại kết nối: tải model (người dùng bấm), remote job (người dùng xác nhận theo từng job), kiểm tra cập nhật (V1, có thể tắt) |
| Consent theo job (BR-04) | Dialog hiển thị: số ảnh, dung lượng, đích (tài khoản Kaggle của người dùng), thời gian lưu, cách xóa |
| Data minimization | Gói remote chỉ chứa ảnh trong scope; file được đổi tên thành `image_id`; không kèm đường dẫn gốc, annotation hay metadata thừa |
| Quyền xóa | Dọn kernel/dataset remote sau khi import (nếu CLI hỗ trợ); nếu không thì hướng dẫn xóa thủ công ([09 §5](../02-kien-truc/09_kaggle_remote_worker.md)) |
| Dữ liệu nhạy cảm trong log | Mặc định redact token; có tùy chọn ẩn đường dẫn home |

## 2. Secret

| Secret | Lưu ở đâu | Không bao giờ |
|---|---|---|
| Kaggle API token | Vị trí chuẩn của Kaggle CLI (`KAGGLE_API_TOKEN`, `~/.kaggle/access_token`, `kaggle.json` legacy), do người dùng cấu hình | Lưu trong source, project config, log, issue, README, chat |
| Hugging Face token (model gated) | `hf auth login` (cache của HF) hoặc OS keyring (V1) | Đưa vào gói remote hay kernel metadata |
| Chứng chỉ ký số khi build | Máy build / secret của CI | Commit vào repo |

- Truyền token cho subprocess chỉ qua biến môi trường của tiến trình con, không qua tham số dòng lệnh, vì tham số có thể bị nhìn thấy trong danh sách tiến trình.
- Từ bản 2.2.1, `kaggle` CLI tự đặt quyền chặt cho file auth. Phải dùng **≥ 2.2.5**, vì bản này sửa lỗi phụ thuộc certifi cũ có thể làm lộ token khi bị tấn công MITM.

## 3. Threat model (rút gọn)

| Tài sản | Mối đe dọa | Biện pháp |
|---|---|---|
| Dataset của người dùng | Bị upload ngoài ý muốn | Consent theo job, có test luồng remote (SC-05) |
| Kaggle/HF token | Lộ qua log, file cấu hình, gói chẩn đoán | Redact filter, không lưu token trong app (MVP), secret scan trong CI |
| Máy người dùng | Artifact remote độc hại (path traversal khi giải nén, JSON khổng lồ) | Validate path, giới hạn kích thước, chỉ parse JSON theo schema, không bao giờ `eval`/`pickle` |
| Máy người dùng | Checkpoint độc hại (pickle trong `.pt`) | Chỉ tải từ nguồn chính thức đã pin checksum; cảnh báo khi dùng checkpoint custom; ưu tiên `safetensors` khi có; `torch.load(weights_only=True)` khi adapter tự load |
| Máy người dùng | Command injection qua tên file hoặc slug | `subprocess` dùng argument list, `shell=False`, slug lấy theo whitelist ký tự |
| Annotation | Bị mất hoặc hỏng | Transaction, WAL, backup, không ghi đè nhãn human-verified (BR-02) |
| Ảnh nguồn | Bị sửa hoặc xóa | Read-only (BR-06): không có code path nào ghi vào dataset root |

## 4. License của sản phẩm

- Mặc định theo Bước 07: **All Rights Reserved** (chờ chủ dự án xác nhận). License này **chỉ áp dụng cho mã của dự án**, không bao trùm thư viện hay model của bên thứ ba.
- `THIRD_PARTY_NOTICES.md` liệt kê nguồn và license của mọi dependency và model asset.
- Người dùng tự chịu trách nhiệm xác minh quyền sử dụng checkpoint và dữ liệu của mình.

## 5. License bên thứ ba (đã xác minh 2026-11-24)

| Thành phần | License | Tác động |
|---|---|---|
| PySide6 / Qt | LGPL-3.0-only (hoặc GPL), có bản thương mại | Cho phép dùng trong app closed-source nếu liên kết động và người dùng thay được thư viện Qt; đóng gói one-folder giúp đáp ứng điều kiện; phải đính kèm license notice |
| PyTorch | BSD-style | Thoải mái |
| **Ultralytics `ultralytics` + weights YOLO** | **AGPL-3.0**; Ultralytics coi cả model đã train/fine-tune là AGPL; sản phẩm closed-source, công cụ nội bộ thương mại, SaaS hay embedded cần **Enterprise License** (giá theo báo giá) | ⚠️ **Xung đột trực tiếp với "All Rights Reserved"**, xem §6 |
| SAM 2 / 2.1 | Apache 2.0 (code + checkpoint) | Thoải mái, giữ notice |
| SAM 3 / 3.1 | **SAM License** (tùy chỉnh): royalty-free, cho phép phân phối/sửa; cấm dùng cho quân sự/ITAR/vũ khí; có điều khoản không reverse-engineer, indemnify Meta; phải kèm license khi phân phối lại; weights bị gated | Có vẻ cho phép dùng thương mại nhưng **cần xác nhận pháp lý**; không bundle weights |
| DINOv2 | Apache 2.0 (trừ Cell-DINO, XRay-DINO là non-commercial) | Chỉ dùng các biến thể Apache |
| DINOv3 | **DINOv3 License** (tùy chỉnh, cấu trúc giống SAM License; Meta có thể thay đổi điều khoản); gated | Future; phải đọc lại license khi phát hành |
| Hugging Face `transformers` | Apache 2.0 | Thoải mái |
| `kaggle` CLI | Apache 2.0 (UNVERIFIED, cần kiểm tra lại) | — |
| Nuitka (nếu dùng) | AGPLv3+ với runtime exception | Output đã biên dịch được miễn trừ theo exception (cần kiểm tra) |

## 6. Quyết định chặn: chiến lược license cho YOLO (ADR-010 — **BLOCKED**, chờ chủ dự án)

| Phương án | Mô tả | Ưu | Nhược |
|---|---|---|---|
| **A. Mở mã nguồn dự án theo AGPL-3.0** | Toàn bộ sản phẩm là AGPL | Dùng YOLO hợp lệ, không tốn phí | Mất quyền closed-source; phải công bố source khi phân phối |
| **B. Mua Ultralytics Enterprise License** | Giữ closed-source | Dùng YOLO26/YOLO11 đầy đủ | Chi phí theo báo giá; phụ thuộc vào thời hạn license |
| **C. Không dùng YOLO trong bản phân phối** | Thay bằng detector/pose có license phù hợp (ứng viên: RT-DETR, D-FINE, RTMDet/RTMPose — **UNVERIFIED**, phải nghiên cứu) | Giữ closed-source, không tốn phí | Thêm công nghiên cứu và tích hợp; chất lượng và tốc độ phải được đo |
| **D. Dự án chỉ dùng cá nhân / học tập, không phân phối** | Không phát hành ra ngoài | Đơn giản | Hạn chế mục đích của dự án |
| **E. "Plugin do người dùng tự cài"** | App không kèm `ultralytics`; người dùng tự cài | — | ⚠️ Rủi ro pháp lý cao, vì có thể vẫn bị coi là tác phẩm phái sinh. **Không khuyến nghị** nếu chưa có ý kiến pháp lý |

**Đề xuất kỹ thuật:** nếu sản phẩm thực sự cần closed-source thì chọn **B** hoặc **C**; nếu không thì chọn **A** hoặc **D**. Kiến trúc adapter ([05](../02-kien-truc/05_model_and_inference_design.md)) cho phép đổi phương án mà không sửa core. M5 bị chặn cho đến khi có quyết định này.

## 7. Điều khoản của Kaggle (rủi ro)

- AUP của Kaggle cấm lạm dụng tài nguyên, như đào coin hay "server farming". Có nguồn trích Terms giới hạn ở mục đích "internal, personal, non-commercial" (UNVERIFIED, chỉ có từ trích dẫn).
- Việc một sản phẩm (đặc biệt là sản phẩm thương mại) tự động điều khiển kernel Kaggle trên tài khoản của người dùng **cần được xem xét pháp lý** trước khi phát hành.
- Biện pháp kỹ thuật: remote luôn là **tùy chọn**, chạy bằng tài khoản và quyết định của chính người dùng, có cảnh báo về điều khoản, không chạy job song song hàng loạt.

## 8. `.gitignore` bắt buộc

```gitignore
# Secrets
.env
*.env.local
kaggle.json
access_token
*.key
*.pem
# Dữ liệu và model
datasets/
*.pt
*.pth
*.onnx
*.engine
*.safetensors
# Runtime
runtime/
.runtime/
cache/
logs/
*.log
jobs/
exports/
# Build
build/
dist/
*.spec.bak
__pycache__/
.venv/
```

## 9. Kiểm tra trước mỗi commit/release

- Secret scan (`gitleaks` hoặc tương đương), không có file lớn ngoài ý muốn, `pip-audit`.
- Kiểm tra license của mọi dependency mới so với danh sách cho phép: MIT, BSD, Apache-2.0, LGPL (liên kết động), PSF. Dependency có license AGPL/GPL hay tùy chỉnh phải qua review.
