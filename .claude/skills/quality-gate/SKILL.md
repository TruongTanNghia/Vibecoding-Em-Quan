---
name: quality-gate
description: Checklist review và kiểm thử bắt buộc trước khi báo một công việc là hoàn thành — chạy test, lint, type-check, kiểm tra nghiệp vụ, bảo mật, giao diện và tài liệu. Dùng ở cuối mỗi tính năng, sửa lỗi hoặc thay đổi code, và khi người dùng yêu cầu review.
---

# Quality Gate — Kiểm tra trước khi báo "xong"

Chỉ báo hoàn thành khi đã đi qua checklist dưới đây. Mục nào không áp dụng → ghi "N/A". Mục nào chưa làm được → **nói rõ** trong báo cáo, không che giấu.

## 1. Chạy được & kiểm chứng

- [ ] `backend`: `npm run lint && npm run typecheck && npm test` — pass
- [ ] `frontend`: `npm run lint && npm run typecheck && npm test` — pass
- [ ] `npm run build` cả hai phía không lỗi
- [ ] Đã chạy thật luồng chính end-to-end (gọi API hoặc thao tác UI), và ít nhất 2 luồng lỗi
- [ ] Báo cáo ghi lại lệnh đã chạy và kết quả (dán output lỗi nếu có)

## 2. Nghiệp vụ

- [ ] Mọi `BR-xx` trong đặc tả đã được hiện thực và có test
- [ ] Luồng thay thế/lỗi trong đặc tả đều có xử lý & mã lỗi
- [ ] Chuyển trạng thái chỉ theo state machine đã định nghĩa
- [ ] Tiêu chí nghiệm thu (Given/When/Then) đều thỏa

## 3. Backend

- [ ] Mọi input được validate; mọi endpoint có auth + phân quyền + kiểm tra sở hữu
- [ ] Response & lỗi đúng format chuẩn
- [ ] Thao tác nhiều bước dùng transaction; đã nghĩ tới bấm 2 lần / đồng thời
- [ ] Có migration cho thay đổi schema; không sửa migration cũ
- [ ] Không có N+1 query rõ ràng ở danh sách; danh sách có phân trang

## 4. Frontend

- [ ] Đủ trạng thái loading / empty / error / success
- [ ] Dùng design token & component `ui/`, không hard-code màu/khoảng cách
- [ ] Responsive (kiểm tra ~375px và ~1280px), không cuộn ngang
- [ ] Form: validate, hiện lỗi backend, chống submit 2 lần
- [ ] Không còn `console.log`, `any`, code chết

## 5. Bảo mật

- [ ] Không có secret/token/mật khẩu trong code hay log
- [ ] `.env.example` cập nhật nếu thêm biến môi trường
- [ ] Không lộ trường nhạy cảm (`passwordHash`, token) trong response
- [ ] Không render HTML từ người dùng mà không sanitize (`dangerouslySetInnerHTML`)

## 6. Tài liệu

- [ ] Tài liệu liên quan trong `docs/` (xem bảng "thay đổi gì → cập nhật tài liệu nào" của skill `tech-docs`) đã cập nhật
- [ ] `CHANGELOG.md` có mục mới
- [ ] README/setup cập nhật nếu cách chạy thay đổi
- [ ] ADR nếu có quyết định kiến trúc

## 7. Mẫu báo cáo cuối

```
Skills đã đọc: ...
Đã làm: ...
Đã kiểm chứng:
- backend: npm test → 42 passed
- frontend: npm run build → OK
- Thủ công: đăng ký khóa học thành công; khóa đầy → báo COURSE_FULL
Chưa làm / rủi ro: ...
```
