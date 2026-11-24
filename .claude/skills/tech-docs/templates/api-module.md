# API — [Tên module]

> Base URL: `/api/v1` · Auth: `Authorization: Bearer <accessToken>` · Cập nhật: YYYY-MM-DD
> Nghiệp vụ: [docs/business/<feature>.md](../business/<feature>.md)

## Tổng quan
| Method | Endpoint | Quyền | Mô tả |
|--------|----------|-------|-------|
| GET | `/resources` | User | Danh sách |
| POST | `/resources` | Admin | Tạo mới |

---

## `POST /resources`
Mô tả ngắn.

**Quyền:** Admin

**Request body**
| Field | Kiểu | Bắt buộc | Ràng buộc |
|-------|------|:--------:|-----------|
| `name` | string | ✓ | 3–100 ký tự |

```json
{ "name": "Lập trình React" }
```

**Response `201`**
```json
{ "data": { "id": "c1a2...", "name": "Lập trình React", "createdAt": "2026-01-01T00:00:00.000Z" } }
```

**Lỗi**
| HTTP | code | Khi nào |
|------|------|---------|
| 400 | `VALIDATION_ERROR` | Input không hợp lệ |
| 401 | `UNAUTHORIZED` | Chưa đăng nhập |
| 403 | `FORBIDDEN` | Không phải Admin |
| 409 | `ALREADY_EXISTS` | Tên đã tồn tại |

**Ví dụ**
```bash
curl -X POST http://localhost:4000/api/v1/resources \
  -H "Authorization: Bearer $TOKEN" -H "Content-Type: application/json" \
  -d '{"name":"Lập trình React"}'
```
