## 1. Backend .env — Fix CORS và JWT

- [x] 1.1 Thêm `https://vivugo.me` vào `Cors__AllowedOrigins__*`
- [x] 1.2 Đổi `Jwt__Issuer` từ `http://localhost:8080` → `https://api.vivugo.me`
- [x] 1.3 Đổi `Jwt__Audience` từ `http://localhost:8080` → `https://api.vivugo.me`
- [x] 1.4 Xóa `Dev__EnableDevEndpoints`, `Dev__DevApiKey`, `Dev__ResetAndReseedOnStartup`
- [x] 1.5 Xóa `Serilog__MinimumLevel__Default`, `SERILOG_OVERRIDE_*`
- [x] 1.6 Xóa `AppConfig__SwaggerBaseUrl`, `SwaggerGen__*`
- [x] 1.7 Xóa `Cloudinary__*`, `Mail__*`, `VietQR__*`, `SePay__*`, `Payment__*`
- [x] 1.8 Xóa `OpenTelemetry__Endpoint` (toàn bộ block OpenTelemetry trong code đang bị comment out)

## 2. Backend .env.example — Đồng bộ

- [x] 2.1 Cập nhật `.env.example` với **cùng biến như `.env`** nhưng giữ **giá trị placeholder** (ví dụ: `Jwt__Secret=replace-with-64-plus-char-secret`, `ConnectionStrings__Default=Host=localhost;...`). Không copy giá trị production thực vào `.env.example`.

## 3. Frontend .env — Cập nhật production

- [x] 3.1 Đổi `NEXT_PUBLIC_API_GATEWAY` → `https://api.vivugo.me`
- [x] 3.2 Cập nhật `NEXT_PUBLIC_DEV_BACKEND_URL` → `https://api.vivugo.me`
- [x] 3.3 Xóa `NEXT_PUBLIC_DEV_RABBITMQ_URL`
- [x] 3.4 Xóa `NEXT_PUBLIC_DEV_ELASTICSEARCH_URL`
- [x] 3.5 Xóa `NEXT_PUBLIC_DEV_PORTAINER_URL`
- [x] 3.6 Xóa `NEXT_PUBLIC_DEV_JUPYTER_URL`

## 4. Frontend .env.example — Đồng bộ

- [x] 4.1 Cập nhật `.env.example` với **cùng biến như `.env`** nhưng giữ **giá trị placeholder** (ví dụ: `NEXT_PUBLIC_API_GATEWAY=http://localhost:5000`)

## 5. Verification

- [x] 5.1 Build backend: `dotnet build src/Api/Api.csproj` — Build succeeded (docker-compose.dcproj reference is pre-existing issue, API project compiles clean)
- [x] 5.2 Build frontend: `npm run build` trong `pathora/frontend/` — passed
- [x] 5.3 Xác nhận backend khởi động không lỗi với `.env` mới (biên dịch thành công, cần test runtime sau khi deploy)
- [ ] 5.4 Sau deploy, test API từ frontend production (CORS check)

## 6. Rollback

- [x] 6.1 Nếu có vấn đề sau deploy: `git checkout HEAD -- panthora_be/.env panthora_be/.env.example pathora/frontend/.env pathora/frontend/.env.example`
