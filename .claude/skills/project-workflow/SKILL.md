---
name: project-workflow
description: Quy trình điều phối end-to-end của dự án — từ yêu cầu, luồng nghiệp vụ, backend, giao diện FE đến tài liệu kỹ thuật và kiểm tra chất lượng. LUÔN đọc skill này đầu tiên trước khi bắt đầu bất kỳ công việc code, thiết kế hay viết tài liệu nào trong dự án.
---

# Project Workflow — Quy trình điều phối

Skill này là "bản đồ". Nó quyết định làm gì theo thứ tự nào và gọi skill chuyên môn nào.

## 1. Phân loại yêu cầu

Trước tiên xác định yêu cầu thuộc loại nào:

| Loại | Ví dụ | Các bước áp dụng |
|------|-------|------------------|
| **A. Tính năng mới full-stack** | "Thêm chức năng đăng ký khóa học" | Bước 1 → 7 đầy đủ |
| **B. Chỉ backend** | "Thêm API xuất báo cáo" | 1, 2, 3, 5, 6, 7 |
| **C. Chỉ frontend** | "Làm lại trang dashboard" | 1, 4, 6, 7 |
| **D. Sửa lỗi** | "Đăng nhập bị lỗi 500" | Tái hiện → tìm nguyên nhân gốc → sửa → test hồi quy → 7 |
| **E. Tài liệu** | "Viết tài liệu API" | 6, 7 |
| **F. Khởi tạo dự án** | "Dựng khung dự án" | Xem mục 4 |

Nếu yêu cầu mơ hồ ở điểm **ảnh hưởng tới thiết kế** (ai dùng, dữ liệu gì, quy tắc nghiệp vụ nào) → hỏi người dùng. Chi tiết kỹ thuật thông thường → tự quyết theo skill và ghi rõ giả định.

## 2. Quy trình 7 bước cho một tính năng

```
1. Hiểu yêu cầu  →  2. Luồng nghiệp vụ  →  3. Backend  →  4. Frontend
        →  5. Tích hợp & test  →  6. Tài liệu  →  7. Quality gate
```

### Bước 1 — Hiểu yêu cầu
- Tóm tắt lại yêu cầu bằng 2–3 câu.
- Liệt kê: vai trò người dùng (actor), dữ liệu vào/ra, tiêu chí hoàn thành (acceptance criteria).
- Đọc code & tài liệu hiện có liên quan (`docs/`, module cùng domain) trước khi thiết kế mới.

### Bước 2 — Luồng nghiệp vụ → đọc `business-flow`
- Viết đặc tả nghiệp vụ vào `docs/01-tong-quan/business/<feature>.md` theo template.
- Có: use case, luồng chính, luồng thay thế/lỗi, quy tắc nghiệp vụ (BR-xx), state machine nếu có trạng thái.

### Bước 3 — Backend → đọc `backend-api`
- Thiết kế/chỉnh schema CSDL → migration.
- Định nghĩa API contract (endpoint, request, response, mã lỗi) **trước** khi code.
- Code theo lớp: route → controller → service → repository. Quy tắc nghiệp vụ nằm ở service.
- Viết test cho service và API.

### Bước 4 — Frontend → đọc `frontend-ui`
- Dựa trên API contract đã chốt ở bước 3.
- Xác định trang/route, component, state, các trạng thái UI (loading / empty / error / success).
- Dùng design token & component có sẵn, không tự chế style rời rạc.

### Bước 5 — Tích hợp & test
- Chạy BE + FE, kiểm tra luồng chính và ít nhất 2 luồng lỗi end-to-end.
- Chạy toàn bộ test, lint, type-check.

### Bước 6 — Tài liệu → đọc `tech-docs`
- Cập nhật tài liệu API, CHANGELOG, README nếu cách chạy thay đổi, ADR nếu có quyết định kiến trúc.

### Bước 7 — Quality gate → đọc `quality-gate`
- Đi qua checklist. Chỉ báo "hoàn thành" khi đạt. Báo cáo trung thực những gì chưa làm / chưa kiểm chứng.

## 3. Nguyên tắc chung

- **Contract trước, code sau:** API contract và luồng nghiệp vụ được chốt trước khi viết UI.
- **Thay đổi nhỏ, kiểm chứng được:** chia tính năng lớn thành các phần chạy được độc lập.
- **Một nguồn sự thật:** kiểu dữ liệu/validate schema dùng chung khi có thể; quy tắc nghiệp vụ chỉ nằm ở backend service (FE chỉ validate để UX tốt hơn).
- **Không phá vỡ:** đổi API đang dùng → versioning hoặc giữ tương thích ngược, ghi CHANGELOG.
- **Bảo mật mặc định:** không commit secret, validate mọi input, kiểm tra quyền ở mọi endpoint.

## 4. Khởi tạo dự án (loại F)

1. Tạo cấu trúc: `backend/`, `frontend/`, `docs/` theo `CLAUDE.md`.
2. Backend: khởi tạo theo mục "Cấu trúc thư mục" trong `backend-api`, có `/health` endpoint, `.env.example`, script `dev`, `build`, `test`, `lint`.
3. Frontend: khởi tạo theo `frontend-ui`, cấu hình Tailwind + design tokens, layout chính, API client.
4. Docs: theo cấu trúc trong `docs/README.md` (01-tong-quan … 07-van-hanh) và `tech-docs`; thêm `CHANGELOG.md`.
5. Root `README.md`: cách cài đặt và chạy cả hệ thống.

## 5. Mẫu báo cáo khi kết thúc công việc

```
Skills đã đọc: project-workflow, business-flow, backend-api, ...
Đã làm:
- ...
Đã kiểm chứng: (lệnh test/chạy nào, kết quả)
Chưa làm / giả định / rủi ro:
- ...
```
