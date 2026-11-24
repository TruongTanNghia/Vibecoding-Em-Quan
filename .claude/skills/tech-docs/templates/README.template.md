# [Tên dự án]

[1–2 câu mô tả hệ thống làm gì, cho ai.]

## Tính năng chính
- ...

## Công nghệ
| Phần | Công nghệ |
|------|-----------|
| Backend | Node.js, Express, TypeScript, Prisma, PostgreSQL |
| Frontend | React, Vite, TypeScript, TailwindCSS |

## Yêu cầu
- Node.js >= 20
- PostgreSQL >= 15 (hoặc Docker)

## Chạy nhanh

```bash
# 1. Backend
cd backend
cp .env.example .env        # sửa DATABASE_URL, JWT_SECRET
npm install
npx prisma migrate dev
npm run seed
npm run dev                 # http://localhost:4000

# 2. Frontend (terminal khác)
cd frontend
cp .env.example .env
npm install
npm run dev                 # http://localhost:5173
```

Tài khoản mẫu: `admin@example.com` / `Admin@123`

## Lệnh thường dùng
| Lệnh | Mô tả |
|------|-------|
| `npm run dev` | Chạy chế độ phát triển |
| `npm test` | Chạy test |
| `npm run lint` | Kiểm tra code style |
| `npm run build` | Build production |

## Tài liệu
- [Mục lục tài liệu](docs/README.md)
- [Tổng quan & yêu cầu](docs/01-tong-quan/00_project_overview.md)
- [Kiến trúc](docs/02-kien-truc/04_system_architecture.md)
- [Cài đặt chi tiết](docs/05-phat-trien/11_environment_and_setup.md)
- [Đóng gói & phát hành](docs/07-van-hanh/15_packaging_release_and_operations.md)
