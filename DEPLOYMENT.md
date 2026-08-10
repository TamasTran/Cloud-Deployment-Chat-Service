# Thông Tin Deploy — Checkpoint 5

> Điền file này sau khi deploy xong. `pytest tests/test_cp5.py` đọc file này
> để tìm địa chỉ service của bạn và gọi thử.
>
> **Chỉ ghi TÊN biến môi trường, tuyệt đối không dán giá trị token vào đây.**
> Repo này công khai — dán token vào là mất token.

## Thông Tin Học Viên

| Mục | Nội dung |
|-----|----------|
| Họ và tên | Trần Tuấn Anh |
| Mã học viên | 2A202601804 |
| Repo | https://github.com/TamasTran/K4-Day12-2A202601804-TRANTUANANH |

## Service

| Mục | Nội dung |
|-----|----------|
| Public URL | https://day12-chat-83su.onrender.com |
| Platform | Render |
| Ngày deploy | 2026-08-10 |

## Biến Môi Trường Đã Set Trên Cloud

Ghi tên biến và **nguồn giá trị**, không ghi giá trị:

| Biến | Đã set | Ghi chú |
|------|--------|---------|
| `PORT` | ✅ | platform tự gán |
| `API_TOKEN` | ✅ | đặt trong dashboard, không nằm trong repo |
| `REDIS_URL` | ✅ | Render Key Value internal connection string |
| `BUCKET_CAPACITY` | ✅ | 10 |
| `REFILL_PER_MINUTE` | ✅ | 10 |
| `DAILY_BUDGET_USD` | ✅ | 1.0 |
| `LOG_LEVEL` | ✅ | INFO |

## Lệnh Kiểm Tra

```bash
# 1. Liveness — mong đợi 200 {"status":"ok"}
curl.exe -i https://day12-chat-83su.onrender.com/healthz

# 2. Readiness — mong đợi 200 {"status":"ready"} (đã nối được Redis)
curl.exe -i https://day12-chat-83su.onrender.com/readyz

# 3. Không có token — mong đợi 401 kèm header WWW-Authenticate
curl.exe -i -X POST https://day12-chat-83su.onrender.com/chat \
  -H "Content-Type: application/json" \
  -d '{"message":"Hello"}'

# 4. Có token — mong đợi 200 kèm câu trả lời
curl.exe -i -X POST https://day12-chat-83su.onrender.com/chat \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer $API_TOKEN" \
  -H "X-Client-Id: sv-test" \
  -d '{"message":"Deploy là gì?"}'

# 5. Rate limit — gọi 15 lần, những lần cuối phải trả 429
for i in $(seq 1 15); do
  curl.exe -s -o /dev/null -w "%{http_code} " -X POST https://day12-chat-83su.onrender.com/chat \
    -H "Content-Type: application/json" \
    -H "Authorization: Bearer $API_TOKEN" \
    -H "X-Client-Id: sv-test" \
    -d '{"message":"test"}'
done; echo
```

## Tiêu Chí Xác Minh

| Kiểm tra | Kết quả yêu cầu |
|----------|-----------------|
| `GET /healthz` | HTTP 200, `status` là `ok` |
| `GET /readyz` | HTTP 200, `status` là `ready`, Redis kết nối thành công |
| `POST /chat` không có token | HTTP 401 và có header `WWW-Authenticate: Bearer` |
| `POST /chat` có token | HTTP 200 và response có trường `reply` |
| Gọi vượt rate limit | Các request cuối trả HTTP 429 |

Các endpoint công khai được bộ kiểm thử `tests/test_cp5.py` xác minh trực tiếp
qua Public URL. Giá trị `API_TOKEN` chỉ được đọc từ `.env` cục bộ thông qua biến
`DEPLOY_API_TOKEN`, không ghi vào tài liệu này.

## Ảnh Chụp Màn Hình

Đặt ảnh trong thư mục `screenshots/`:

- `screenshots/dashboard.png` — trang quản lý service trên platform
- `screenshots/healthz.png` — kết quả gọi `/healthz` từ trình duyệt hoặc curl

## Trạng Thái Triển Khai

Service được triển khai trực tiếp trên Render, sử dụng Render Key Value làm
Redis. Không sử dụng phương án `LOCAL_FALLBACK`.
