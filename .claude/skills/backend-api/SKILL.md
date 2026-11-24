---
name: backend-api
description: Chuẩn xây dựng hệ thống backend — kiến trúc phân lớp, cấu trúc thư mục, thiết kế CSDL & migration (Prisma/PostgreSQL), REST API contract, validate (Zod), xác thực/phân quyền (JWT), xử lý lỗi, logging, bảo mật và test. Dùng khi tạo hoặc sửa bất kỳ code nào trong backend/, thiết kế bảng dữ liệu hay endpoint API.
---

# Backend API — Chuẩn xây dựng backend

Stack mặc định: Node.js 20+, TypeScript (strict), Express, Prisma, PostgreSQL, Zod, JWT, Vitest + Supertest.
(Nếu `CLAUDE.md` khai báo stack khác, giữ nguyên **nguyên tắc** và đổi công cụ tương ứng.)

## 1. Cấu trúc thư mục

```
backend/
├── prisma/
│   ├── schema.prisma
│   ├── migrations/
│   └── seed.ts
├── src/
│   ├── app.ts                 # Tạo Express app, gắn middleware, routes
│   ├── server.ts              # Khởi động server
│   ├── config/
│   │   └── env.ts             # Đọc & validate biến môi trường bằng Zod
│   ├── lib/
│   │   ├── prisma.ts          # Prisma client singleton
│   │   ├── logger.ts
│   │   └── errors.ts          # AppError + các lỗi chuẩn
│   ├── middlewares/
│   │   ├── auth.ts            # requireAuth, requireRole
│   │   ├── validate.ts        # validate(schema) cho body/query/params
│   │   └── error-handler.ts
│   ├── modules/               # Chia theo domain nghiệp vụ
│   │   └── <module>/
│   │       ├── <module>.routes.ts
│   │       ├── <module>.controller.ts
│   │       ├── <module>.service.ts
│   │       ├── <module>.repository.ts
│   │       ├── <module>.schema.ts      # Zod schema + type request/response
│   │       └── <module>.test.ts
│   └── routes.ts              # Gom routes: /api/v1/...
├── .env.example
├── package.json
└── tsconfig.json
```

## 2. Phân lớp & trách nhiệm

| Lớp | Làm gì | KHÔNG làm gì |
|-----|--------|--------------|
| **routes** | Khai báo path, gắn middleware auth/validate | Logic |
| **controller** | Lấy input đã validate, gọi service, trả response | Truy vấn DB, quy tắc nghiệp vụ |
| **service** | Quy tắc nghiệp vụ (BR-xx), transaction, điều phối | Biết về `req`/`res` |
| **repository** | Truy vấn Prisma | Quy tắc nghiệp vụ |

Luồng: `request → auth → validate → controller → service → repository → DB`.

## 3. Thiết kế CSDL

- Bảng: `snake_case` số nhiều qua `@@map("courses")`; model Prisma `PascalCase` số ít.
- Mọi bảng có: `id` (UUID hoặc cuid), `createdAt`, `updatedAt`. Xóa mềm khi nghiệp vụ cần: `deletedAt`.
- Trạng thái dùng `enum`. Tiền dùng `Decimal` hoặc `Int` (đơn vị nhỏ nhất). Thời gian lưu UTC.
- Đánh index cho khóa ngoại và trường hay lọc/sắp xếp; `@@unique` cho ràng buộc nghiệp vụ (vd: 1 học viên chỉ đăng ký 1 lần/khóa).
- Thay đổi schema → **luôn** tạo migration (`npx prisma migrate dev --name <mo_ta>`), không sửa migration đã chạy.
- Cập nhật sơ đồ ERD (Mermaid `erDiagram`) trong `docs/architecture.md` khi đổi schema.

## 4. API contract

- Prefix: `/api/v1`. Resource danh từ số nhiều, `kebab-case`: `/api/v1/course-categories`.
- Method: `GET` đọc, `POST` tạo, `PATCH` sửa một phần, `PUT` thay thế, `DELETE` xóa. Hành động nghiệp vụ không phải CRUD: `POST /api/v1/enrollments/:id/cancel`.
- JSON field: `camelCase`.

**Response thành công:**
```json
{ "data": { ... } }
{ "data": [ ... ], "meta": { "page": 1, "pageSize": 20, "total": 135 } }
```

**Response lỗi (thống nhất toàn hệ thống):**
```json
{ "error": { "code": "COURSE_FULL", "message": "Khóa học đã hết chỗ", "details": [] } }
```

| HTTP | Khi nào | code ví dụ |
|------|---------|------------|
| 400 | Input sai định dạng | `VALIDATION_ERROR` |
| 401 | Chưa đăng nhập / token sai | `UNAUTHORIZED` |
| 403 | Không đủ quyền | `FORBIDDEN` |
| 404 | Không tìm thấy | `NOT_FOUND` |
| 409 | Xung đột trạng thái / trùng lặp | `ALREADY_EXISTS`, `INVALID_STATE_TRANSITION` |
| 422 | Vi phạm quy tắc nghiệp vụ | `COURSE_FULL` |
| 500 | Lỗi không lường trước | `INTERNAL_ERROR` (không lộ stack trace) |

- Danh sách: hỗ trợ `?page=&pageSize=&sort=&q=`; `pageSize` tối đa 100.
- Ghi contract vào `docs/api/<module>.md` (hoặc OpenAPI) **trước** khi FE dùng.

## 5. Validate

- Mọi input (`body`, `query`, `params`) validate bằng Zod qua middleware `validate()`.
- Export type từ schema: `export type CreateCourseInput = z.infer<typeof createCourseSchema>`.
- Biến môi trường validate trong `config/env.ts` — thiếu biến bắt buộc thì app không khởi động.

## 6. Xử lý lỗi

```ts
// lib/errors.ts
export class AppError extends Error {
  constructor(
    public status: number,
    public code: string,
    message: string,
    public details?: unknown,
  ) { super(message); }
}
```
- Service `throw new AppError(422, 'COURSE_FULL', 'Khóa học đã hết chỗ')`.
- Một `error-handler` middleware duy nhất chuyển lỗi → response chuẩn; lỗi Zod → 400; lỗi Prisma `P2002` → 409; còn lại → 500 + log.
- Controller async được bọc để lỗi luôn tới error-handler (Express 5 hoặc wrapper `asyncHandler`).

## 7. Xác thực & phân quyền

- Đăng nhập trả **access token** JWT ngắn hạn (15 phút) + **refresh token** dài hạn (httpOnly cookie, lưu hash trong DB để thu hồi được).
- Mật khẩu hash bằng `bcrypt` (cost ≥ 10) hoặc `argon2`. Không bao giờ trả `passwordHash` ra API.
- Middleware `requireAuth` → gắn `req.user`; `requireRole('ADMIN')` cho phân quyền theo vai trò.
- Kiểm tra **quyền sở hữu** trong service (user chỉ sửa dữ liệu của mình) — không chỉ dựa vào role.

## 8. Bảo mật

- `helmet`, CORS whitelist theo env, rate limit cho `/auth/*`.
- Không log mật khẩu, token, dữ liệu nhạy cảm.
- Không commit `.env`; luôn cập nhật `.env.example`.
- Chỉ dùng truy vấn Prisma có tham số; nếu `$queryRaw` thì dùng tagged template, không nối chuỗi.

## 9. Transaction & đồng thời

- Thao tác nhiều bảng hoặc kiểm tra-rồi-ghi (vd: còn chỗ → tạo đăng ký) → `prisma.$transaction`.
- Tránh race condition bằng ràng buộc unique, update có điều kiện (`updateMany where: { status: 'PENDING' }`) hoặc khóa dòng.

## 10. Logging

- Logger có cấu trúc (pino). Mỗi request có `requestId`. Log level theo env.
- Log lỗi 5xx kèm stack; lỗi 4xx chỉ log mức `warn`/`info`.

## 11. Test

- **Service test:** mỗi quy tắc `BR-xx` có ít nhất 1 test đúng + 1 test vi phạm. Tên test chứa mã BR.
- **API test (Supertest):** happy path, 400, 401/403, 404 cho mỗi endpoint.
- DB test riêng (`DATABASE_URL` test), reset dữ liệu giữa các test.
- Script: `npm run dev | build | start | test | lint | typecheck`.

## 12. Checklist khi thêm một module

- [ ] Schema Prisma + migration
- [ ] `<module>.schema.ts` (Zod) cho mọi input
- [ ] repository → service (có BR) → controller → routes, đã gắn vào `routes.ts`
- [ ] Auth + phân quyền + kiểm tra sở hữu
- [ ] Mã lỗi cụ thể cho từng luồng lỗi
- [ ] Test service + API, tất cả pass
- [ ] Tài liệu API trong `docs/api/`
