---
name: frontend-ui
description: Chuẩn thiết kế và xây dựng giao diện frontend — design system (màu, typography, spacing), cấu trúc thư mục React, component, routing, quản lý state, gọi API (TanStack Query), form & validate, các trạng thái loading/empty/error, responsive, accessibility. Dùng khi tạo hoặc sửa trang, component, layout, style hay bất kỳ code nào trong frontend/.
---

# Frontend UI — Chuẩn thiết kế & xây dựng giao diện

Stack mặc định: React 18, Vite, TypeScript (strict), TailwindCSS, React Router, TanStack Query, React Hook Form + Zod, Vitest + Testing Library.

## 1. Cấu trúc thư mục

```
frontend/
├── src/
│   ├── main.tsx
│   ├── app/
│   │   ├── router.tsx          # Khai báo route
│   │   ├── providers.tsx       # QueryClient, Auth, Theme...
│   │   └── layouts/            # MainLayout, AuthLayout, AdminLayout
│   ├── components/
│   │   └── ui/                 # Component dùng chung: Button, Input, Modal, Table, Badge, Toast...
│   ├── features/               # Chia theo domain nghiệp vụ (khớp modules/ ở backend)
│   │   └── <feature>/
│   │       ├── api.ts          # Hàm gọi API + query keys
│   │       ├── hooks.ts        # useCourses, useCreateCourse...
│   │       ├── types.ts
│   │       ├── schema.ts       # Zod schema cho form
│   │       ├── components/
│   │       └── pages/
│   ├── lib/
│   │   ├── api-client.ts       # fetch wrapper: baseURL, token, xử lý lỗi chuẩn
│   │   └── utils.ts            # cn(), formatDate, formatCurrency...
│   └── styles/
│       └── index.css           # Tailwind + CSS variables (design tokens)
├── .env.example                # VITE_API_URL=...
└── tailwind.config.ts
```

## 2. Design system

**Nguyên tắc:** giao diện nhất quán, gọn gàng, dễ đọc; mọi màu/khoảng cách/bo góc lấy từ token — **không** hard-code mã màu hay giá trị lẻ trong component.

**Design tokens** (CSS variables trong `styles/index.css`, ánh xạ vào `tailwind.config.ts`):

| Nhóm | Token | Gợi ý giá trị |
|------|-------|---------------|
| Màu nền | `--bg`, `--surface`, `--surface-muted` | trắng / xám rất nhạt |
| Chữ | `--text`, `--text-muted` | tương phản ≥ 4.5:1 |
| Thương hiệu | `--primary`, `--primary-foreground` | 1 màu chủ đạo |
| Trạng thái | `--success`, `--warning`, `--danger`, `--info` | |
| Viền | `--border` | |
| Bo góc | `--radius` | 8px |

- Hỗ trợ dark mode bằng cách định nghĩa lại token dưới `.dark` (hoặc `prefers-color-scheme`).
- **Typography:** 1 font sans chính; thang cỡ chữ `text-xs` → `text-3xl`; tiêu đề `font-semibold`; độ dài dòng văn bản ≤ ~75 ký tự.
- **Spacing:** dùng thang 4px của Tailwind (`p-2, p-4, gap-6`...). Không dùng giá trị tùy ý `p-[13px]`.
- **Icon:** 1 bộ duy nhất (vd: `lucide-react`), kích thước 16/20/24.
- Trước khi tạo component mới → kiểm tra `components/ui/` đã có chưa. Component UI mới phải hỗ trợ `variant`, `size`, `disabled`, và `className` để mở rộng.
- Khi cần thiết kế giao diện có tính thẩm mỹ cao (landing page, dashboard mới), có thể kết hợp skill `frontend-design`.

## 3. Bố cục & responsive

- Mobile-first. Breakpoint: `sm 640`, `md 768`, `lg 1024`, `xl 1280`.
- Nội dung chính trong container tối đa `max-w-7xl mx-auto px-4`.
- Bảng dữ liệu trên mobile → cuộn ngang trong vùng riêng hoặc chuyển dạng card. Trang không bao giờ bị cuộn ngang.
- Vùng bấm tối thiểu 40×40px.

## 4. Mỗi màn hình phải có đủ 4 trạng thái

| Trạng thái | Hiển thị |
|------------|----------|
| **Loading** | Skeleton (ưu tiên) hoặc spinner; không nhảy layout |
| **Empty** | Thông điệp + hành động gợi ý ("Chưa có khóa học nào — Tạo khóa học") |
| **Error** | Thông điệp dễ hiểu + nút "Thử lại"; không hiện lỗi kỹ thuật thô |
| **Success** | Dữ liệu; sau hành động ghi → toast xác nhận |

## 5. Gọi API & state

- **Server state** → TanStack Query. Không lưu dữ liệu từ API vào `useState`/global store.
- **UI state** cục bộ → `useState`. State dùng chung (auth, theme) → Context hoặc Zustand.
- Query key có cấu trúc: `['courses', 'list', params]`, `['courses', 'detail', id]`. Mutation xong → `invalidateQueries` key liên quan.
- `api-client.ts` là điểm duy nhất gọi `fetch`: gắn token, parse lỗi chuẩn `{ error: { code, message } }` thành `ApiError`, tự refresh token khi 401 (1 lần), rồi chuyển về đăng nhập nếu thất bại.
- Kiểu dữ liệu request/response khớp với contract trong `docs/api/`.
- Hiển thị thông báo lỗi theo `error.code` (map sang câu tiếng Việt thân thiện) thay vì hiện `message` thô khi cần.

## 6. Form

- React Hook Form + Zod resolver. Schema validate FE nên khớp với backend (backend vẫn là nơi kiểm tra cuối cùng).
- Lỗi hiển thị ngay dưới field; label luôn có (không chỉ dùng placeholder).
- Nút submit disabled + hiện loading khi đang gửi; chống gửi 2 lần.
- Lỗi `VALIDATION_ERROR` từ backend → gán vào field tương ứng qua `setError`.
- Hành động nguy hiểm (xóa, hủy) → modal xác nhận.

## 7. Routing & phân quyền

- Route được bảo vệ bọc trong `<RequireAuth roles={[...]}>`. Ẩn nút/menu mà user không có quyền — nhưng **backend vẫn là nơi chặn thật**.
- Trang lớn lazy-load (`React.lazy`). Có trang 404.
- Lọc/phân trang/tìm kiếm của danh sách đồng bộ lên URL query (`?page=2&q=react`) để chia sẻ link và F5 không mất.

## 8. Accessibility

- Dùng thẻ ngữ nghĩa (`button`, `nav`, `main`, `label`). Không dùng `div onClick`.
- Mọi ảnh có `alt`; icon-only button có `aria-label`.
- Điều hướng được bằng bàn phím; focus ring rõ ràng (`focus-visible:ring-2`).
- Modal: bẫy focus, đóng bằng Esc, trả focus khi đóng.

## 9. Định dạng hiển thị (Việt Nam)

- Ngày: `dd/MM/yyyy`, giờ `HH:mm`, múi giờ `Asia/Ho_Chi_Minh`.
- Tiền: `Intl.NumberFormat('vi-VN', { style: 'currency', currency: 'VND' })`.
- Text giao diện tiếng Việt; gom vào hằng số/từ điển nếu dự án cần đa ngôn ngữ.

## 10. Quy ước code

- Component: `PascalCase.tsx`, 1 component chính/file, tối đa ~200 dòng — dài hơn thì tách.
- Hook: `useXxx`. Không gọi API trực tiếp trong component — qua hook của feature.
- Không dùng `any`. Props có type rõ ràng.
- Test: component quan trọng & form có test hành vi (render, nhập, submit, hiển thị lỗi).

## 11. Checklist khi làm một màn hình

- [ ] Route + layout + phân quyền
- [ ] Dùng component `ui/` và design token, không style lẻ
- [ ] Đủ 4 trạng thái loading / empty / error / success
- [ ] Gọi API qua hook TanStack Query, invalidate sau mutation
- [ ] Form validate + lỗi từ backend + chống submit 2 lần
- [ ] Responsive mobile → desktop, không cuộn ngang
- [ ] Bàn phím & aria cơ bản
- [ ] `npm run lint`, `typecheck`, `test` pass
