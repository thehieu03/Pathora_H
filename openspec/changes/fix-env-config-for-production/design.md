## Context

Dự án có hai service deploy riêng:
- **Backend**: `https://api.vivugo.me` (ASP.NET Core, port 8080)
- **Frontend**: `https://vivugo.me/` (Next.js, port 3000)

Frontend call API cross-origin nên cần CORS. JWT tokens cần có issuer/audience match với production domain.

## Goals / Non-Goals

**Goals:**
- CORS backend cho phép `https://vivugo.me`
- JWT issuer/audience production-ready (`https://api.vivugo.me`)
- `.env` và `.env.example` chỉ chứa biến thực sự được code đọc
- Frontend API gateway trỏ đúng đến backend production

**Non-Goals:**
- Không đổi cấu trúc code, không thêm feature
- Không migration database
- Không thay đổi Docker Compose (đã có sẵn)

## Decisions

### 1. Backend JWT Issuer/Audience
- **Chọn**: Sử dụng `https://api.vivugo.me` cho cả `Jwt__Issuer` và `Jwt__Audience`
- **Alternatives**: Dùng `*` hoặc disable validation — không an toàn, không làm
- **Lý do**: Production HTTPS domain là giá trị ổn định, không phụ thuộc port/path

### 2. CORS AllowedOrigins
- **Chọn**: Thêm `https://vivugo.me` vào danh sách CORS origins
- **Giữ lại**: `localhost` origins vì dev workflow vẫn cần

### 3. Environment Variables — Chỉ giữ biến thật sự đọc
Sau khi grep toàn bộ code `panthora_be/src/`:
- Đọc trực tiếp: `Auth__DisableAuthorization`, `ASPNETCORE_ENVIRONMENT`, `ConnectionStrings__Default`, `Redis__ConnectionString`, `Jwt__Secret`, `Jwt__Issuer`, `Jwt__Audience`, `Authentication__Google__*`, `Cors__AllowedOrigins__*`, `RateLimit__*`, `OutboxWorker__*`, `OpenTelemetry__Endpoint`
- Đọc qua Options classes: `Jwt__ExpireInMinutes`, `Jwt__*Cookie*`, `Jwt__RefreshTokenExpirationHours`
- **Xóa**: `Dev__*`, `Serilog__*` (dùng appsettings.json), `AppConfig__SwaggerBaseUrl`, `SwaggerGen__*`, `Cloudinary__*`, `Mail__*`, `VietQR__*`, `SePay__*`, `Payment__*` (code không đọc)

### 3b. `.env.example` vs `.env` — Phân biệt rõ ràng
- **`.env`** — chứa giá trị thực (hoặc giá trị mặc định cho môi trường hiện tại)
- **`.env.example`** — chứa **placeholder values** cùng biến với `.env`, giá trị mẫu không phải thật (ví dụ: `replace-with-db-password`, `replace-with-64-plus-char-secret`)
- Lý do: `.env.example` được commit vào git, không chứa secrets thực. Người mới copy file này và thay giá trị.

### 4. Frontend — DEV URLs
- Giữ nguyên `DEV_*` URLs vì chúng dùng cho admin dashboard dev links (localhost)
- Xóa `NEXT_PUBLIC_DEV_RABBITMQ_URL`, `ELASTICSEARCH_URL`, `PORTAINER_URL`, `JUPYTER_URL` (code không dùng)

## Risks / Trade-offs

- **[Risk] JWT token hết hiệu lực sau khi deploy** → Đây là behavior bình thường, user cần re-login
- **[Risk] Quên update CORS khi thêm domain mới** → Thêm comment trong .env hướng dẫn
- **[Trade-off] Xóa biến thừa** → Giảm confusion, nhưng nếu có service mới enable sau sẽ cần thêm lại
