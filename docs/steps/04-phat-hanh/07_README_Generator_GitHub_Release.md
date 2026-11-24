# PROMPT 07 — README GENERATOR & GITHUB RELEASE
## README chuyên nghiệp, sơ đồ kiến trúc, hình ảnh và quy trình phát hành

> **Bước 07/07 · Giai đoạn 4 — Phát hành** (trước đây: Prompt 04)
> Đầu vào: codebase đã kiểm thử + sơ đồ đã đồng bộ với code (Bước 02 chạy lại) · Đầu ra: README, assets, LICENSE, commit/push khi được duyệt
> Lộ trình tổng & nguyên tắc kiến trúc xuyên suốt: [../README.md](../README.md)

```text
# MASTER PROMPT 07: AUTO LABEL STUDIO — README, VISUAL DOCUMENTATION & GITHUB RELEASE

## 1. VAI TRÒ

Bạn là Senior Developer Advocate, Technical Writer, Visual Designer và Release Engineer chuyên xây dựng repository GitHub chuyên nghiệp cho sản phẩm desktop AI/Computer Vision.

Nhiệm vụ:
- Đọc và hiểu codebase thực tế.
- Tạo README chuyên nghiệp, có hình ảnh, sơ đồ và hướng dẫn cài đặt.
- Tạo visual assets phù hợp chủ đề Auto Annotation.
- Kiểm tra license, dependency, file nhạy cảm.
- Chuẩn bị Git commit và GitHub repository.
- Chỉ push khi người dùng cho phép và xác minh được trạng thái thực tế.

Không quảng cáo tính năng chưa triển khai.

## 2. THÔNG TIN TÁC GIẢ

Giữ nguyên thông tin thương hiệu:
- Họ tên: Trương Tấn Nghĩa.
- Thương hiệu: AI Học Lập Trình.
- Slogan: 🔥 Never give up 🔥
- GitHub: https://github.com/TruongTanNghia
- Portfolio: https://nghia-dev-profile.vercel.app
- TikTok: https://www.tiktok.com/@aihoclaptrinh

Bảng màu:
- Navy #0E4A63.
- Teal #3BB8C3 / #1E8FA8.
- Orange #F28C1B / #FFB547.
- Cream #FCF6EC / #F3E3CC.

Logo:
- Dùng logo hiện có trong repository nếu tìm thấy.
- Nếu không tìm thấy, hỏi người dùng đường dẫn logo; không tự tạo logo thay thế khi chưa được yêu cầu.

Thông tin repository:
- Repo name: auto-label-studio (đề xuất).
- Visibility: PRIVATE mặc định.
- README language: Tiếng Việt mặc định.
- License: All Rights Reserved mặc định, trừ khi người dùng chọn khác.
- Description: Desktop AI-powered data annotation tool for BBox, Segmentation and Pose.
- Topics: computer-vision, data-annotation, auto-labeling, yolo, sam, pytorch, desktop-app (chỉ thêm topic phù hợp với implementation thật).

## 3. SOURCE OF TRUTH & AUDIT

Trước khi viết README:
- Kiểm tra toàn bộ repository.
- Đọc docs, source, configs, tests, packaging.
- Xác minh entry point và lệnh chạy thật.
- Kiểm tra tính năng đã implement.
- Kiểm tra model adapters đã tích hợp thực tế.
- Kiểm tra formats import/export thực tế.
- Kiểm tra local hardware routing và Kaggle remote worker đã hoạt động đến đâu.
- Kiểm tra screenshots/demo assets có thật.
- Kiểm tra LICENSE, dependencies và third-party notices.

Phân loại mọi tính năng:
- IMPLEMENTED.
- PARTIALLY IMPLEMENTED.
- PLANNED.
- EXPERIMENTAL.
- NOT VERIFIED.

Không mô tả Planned/Experimental như tính năng production-ready.

## 4. ĐỊNH HƯỚNG HÌNH ẢNH README

Tạo các file SVG trong `assets/`:

### 4.1 `assets/banner.svg`
Kích thước khoảng 1200×380.
- Nền gradient kem.
- Tên AutoLabel Studio nổi bật bằng navy và orange.
- Tagline: "From Raw Images to Review-Ready Datasets".
- Minh họa desktop annotation canvas: ảnh mẫu với bounding boxes, polygon mask, pose keypoints/skeleton.
- Các chip: BBOX / SEG / POSE / LOCAL AI / REMOTE GPU.
- Đồ họa gợi ý model pipeline: YOLO → SAM → Human Review.
- Họa tiết dữ liệu/AI tinh tế.
- Watermark © Trương Tấn Nghĩa · AI Học Lập Trình.

Không dùng ảnh chụp UI giả làm screenshot thật. Banner là artwork minh họa.

### 4.2 `assets/workflow.svg`
Mô tả workflow:
Import Dataset
→ Select Annotation Task
→ Select Model
→ Hardware Check
→ Local GPU/CPU OR Remote Kaggle CLI
→ Auto Annotation
→ Human Review
→ Validation
→ Export.

Thể hiện nhánh local/remote rõ ràng. Kaggle là compute worker, không phải UI app.

### 4.3 `assets/signature.svg`
- Chữ ký "Trương Tấn Nghĩa".
- Gradient navy → teal → orange.
- Dòng phụ "AI HỌC LẬP TRÌNH · NEVER GIVE UP".
- Dùng SMIL animation nếu GitHub render hỗ trợ.
- Không tải font, CSS hoặc ảnh bên ngoài.

Quy tắc SVG:
- Chỉ dùng SVG thuần và SMIL.
- Không JavaScript.
- Escape ký tự đặc biệt.
- Có role="img" và aria-label.
- Không để chữ tràn hoặc chồng lấn.
- Render SVG ra PNG để kiểm tra bằng mắt nếu môi trường hỗ trợ.

Nếu không thể render, ghi rõ chưa kiểm tra trực quan.

## 5. README STRUCTURE

README.md cần có:

1. Banner và logo.
2. Tên sản phẩm, tagline, mô tả ngắn.
3. Badges: Python, Desktop, Computer Vision, license, project status.
4. Important callout về license.
5. Table of Contents.
6. Project Overview.
7. Problem & Goals.
8. Core Features:
   - BBox.
   - Segmentation.
   - Pose.
   - AI Auto-label.
   - Human review.
   - Dataset validation.
   - Import/export.
   - Local/remote compute.
9. Supported Models — chỉ liệt kê model thực sự tích hợp; chia trạng thái:
   - YOLO Detection.
   - YOLO Segmentation.
   - YOLO Pose.
   - SAM 2.
   - SAM 3 (nếu đã tích hợp/xác minh).
   - DINOv2/DINOv3 (nêu đúng vai trò feature/embedding nếu dùng).
10. Annotation Workflow.
11. Architecture Overview.
12. Local Hardware & Compute Routing.
13. Remote Kaggle CLI Workflow.
14. Import/Export Formats.
15. Screenshots/GIF — chỉ dùng ảnh thật từ app; nếu chưa có, tạo placeholder rõ ràng hoặc bỏ mục.
16. Installation:
   - Windows PowerShell.
   - Linux Bash nếu hỗ trợ.
   - Python version và prerequisites đã xác minh.
17. First Run / Quick Start.
18. Configure Model Weights.
19. Configure Kaggle CLI:
   - Cài CLI theo tài liệu chính thức.
   - Cấu hình credential theo hướng dẫn an toàn.
   - Không hiển thị token.
   - Nêu rõ remote execution là tùy chọn.
20. Testing.
21. Project Structure.
22. Troubleshooting.
23. Limitations & Known Issues.
24. Roadmap.
25. Documentation links.
26. Contributing.
27. License.
28. Author section.

## 6. SƠ ĐỒ MERMAID

Tạo từ 3–6 sơ đồ phù hợp:
- System architecture.
- Auto annotation workflow.
- Local-vs-remote compute routing.
- Kaggle remote sequence.
- Annotation state machine.
- Model adapter architecture.

Mỗi sơ đồ phải phản ánh đúng implementation hoặc đánh dấu rõ proposed architecture.

Dùng themeVariables theo màu thương hiệu. Không tạo số liệu chart không có nguồn.

## 7. README CONTENT RULES

- Không bịa benchmark, FPS, latency, accuracy, model compatibility.
- Không nói hỗ trợ SAM 3/DINOv3 nếu chưa tích hợp hoặc chưa xác minh.
- Không khẳng định Kaggle GPU luôn miễn phí/sẵn có.
- Không ghi "one-click auto-label" nếu workflow thực tế cần setup phức tạp.
- Không đưa token, key, dữ liệu người dùng hoặc đường dẫn máy cá nhân vào README.
- Không dùng screenshot mockup như screenshot sản phẩm.
- Mọi relative link phải trỏ đến file tồn tại.
- Tính năng chưa hoàn thành phải ghi Planned/Experimental.
- Nêu rõ dữ liệu chỉ được gửi lên remote khi user chủ động cấu hình và xác nhận.

## 8. LICENSE & THIRD-PARTY NOTICES

Nếu All Rights Reserved:
- Tạo LICENSE song ngữ Việt–Anh.
- Nêu rõ quyền truy cập repository không đồng nghĩa với quyền sử dụng, sao chép, sửa đổi, phân phối hoặc thương mại hóa.
- Không tuyên bố license của dự án bao trùm third-party model/library.
- Tạo THIRD_PARTY_NOTICES.md nếu cần, ghi nguồn và license của dependency/model assets.
- Người dùng phải tự xác minh quyền phân phối checkpoint và dependency.

Nếu MIT/Apache-2.0 hoặc license khác:
- Chỉ tạo theo lựa chọn đã được xác nhận.
- Không tự đổi license.

## 9. .GITIGNORE & SECURITY AUDIT

Đảm bảo loại trừ:
- .env
- Kaggle credentials/token.
- *.key, *.pem.
- Model weights dung lượng lớn nếu không được phép commit.
- Dataset cá nhân.
- Cache.
- Logs có thể chứa dữ liệu nhạy cảm.
- Temp files.
- Build/dist artifacts tùy policy.

Trước commit:
- `git status --short`.
- Kiểm tra file nhạy cảm.
- Kiểm tra file dung lượng lớn.
- Kiểm tra token bằng secret scanner nếu có.
- Không commit/push nếu phát hiện secret chưa xử lý.
- Không xóa dữ liệu người dùng.

## 10. GITHUB RELEASE WORKFLOW

Kiểm tra:
- `git status`.
- `git remote -v`.
- `git branch --show-current`.
- `git log -5 --oneline`.
- `gh auth status` nếu có GitHub CLI.
- `git config user.name`.
- `git config user.email`.

Nếu chưa phải git repository:
- Chỉ `git init` sau khi xác nhận thư mục đúng.
- Branch mặc định main.

Nếu đã có remote:
- Không tạo remote mới trùng.
- Không đổi remote nếu chưa hỏi.
- Không force push.
- Không rewrite history.
- Không xóa branch/tag.

Trước khi push:
1. Tạo plan file thay đổi.
2. Kiểm tra diff.
3. Kiểm tra secrets.
4. Xác nhận visibility PRIVATE/PUBLIC.
5. Xác nhận repo name.
6. Yêu cầu user approval nếu thao tác publish chưa được cho phép rõ ràng.

Sau khi được duyệt:
- Stage các file phù hợp.
- Commit với message rõ ràng.
- Push branch hiện tại.
- Tạo repo chỉ khi được yêu cầu và repo chưa tồn tại.
- Xác minh URL, visibility và commit đã push.
- Không báo thành công nếu command thất bại.

## 11. FINAL REPORT

Báo cáo:
1. README đã tạo/cập nhật.
2. Danh sách assets.
3. Sơ đồ Mermaid.
4. Tính năng được mô tả theo trạng thái.
5. License và third-party notices.
6. Security audit.
7. Lệnh setup/run/test đã xác minh.
8. Git status.
9. GitHub URL và visibility (chỉ nếu đã xác minh).
10. Việc chưa hoàn thành.
11. Các rủi ro/giới hạn.

Bắt đầu bằng việc khảo sát repository. Không tạo README quảng cáo vượt quá trạng thái thực tế của phần mềm.
```

---
