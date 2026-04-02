## Why

Backend đã deploy lên `https://api.vivugo.me` và frontend sắp deploy lên `https://vivugo.me/`, nhưng cấu hình environment hiện tại chưa được cập nhật cho production. JWT issuer/audience vẫn trỏ về `localhost`, CORS thiếu production domain, và cả hai `.env` đều chứa nhiều biến không được code sử dụng.

## What Changes

- **Backend `.env`**: Thêm CORS production domain (`https://vivugo.me`), sửa JWT issuer/audience thành `https://api.vivugo.me`, xóa các biến thừa không dùng đến (Serilog overrides, SwaggerGen, Cloudinary, Mail, VietQR, SePay, Payment configs, OpenTelemetry)
- **Frontend `.env`**: Cập nhật `NEXT_PUBLIC_API_GATEWAY` thành `https://api.vivugo.me`, xóa các biến DEV không dùng (RabbitMQ, Elasticsearch, Portainer, Jupyter)
- **Backend `.env.example`**: Đồng bộ với `.env` thực tế — chỉ giữ lại các biến thật sự được code đọc

## Capabilities

### New Capabilities
- `env-config-production`: Dọn dẹp và cập nhật cấu hình environment cho production deployment

### Modified Capabilities
*(None — đây là thay đổi config, không thay đổi requirement của system)*

## Impact

- **Backend**: `panthora_be/.env`, `panthora_be/.env.example`
- **Frontend**: `pathora/frontend/.env`, `pathora/frontend/.env.example`
- **Không ảnh hưởng** API contracts, database schema, hay business logic
