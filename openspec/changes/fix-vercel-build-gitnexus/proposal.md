## Why

Vercel build thất bại khi deploy `pathora/` vì `gitnexus@1.4.9` nằm trong `dependencies` của `pathora/package.json`. `gitnexus` là native addon (Node.js C++) phụ thuộc vào `tree-sitter`, yêu cầu C++20 headers từ V8/Node 24. Vercel build machine (Node 24.14.1 + gcc/Linux) không hỗ trợ C++20 đầy đủ, dẫn đến `node-gyp` build fail với hàng trăm lỗi compile (`#error "C++20 or later required."`). Trên máy local (Windows/MSVC), `gitnexus` có sẵn pre-built binary nên không gặp lỗi.

## What Changes

- Remove `gitnexus` khỏi `pathora/package.json` — chỉ cần cho local development, không cần trên Vercel
- Hoặc chuyển `gitnexus` từ `dependencies` sang `devDependencies`
- Cập nhật `vercel.json` để skip install devDependencies khi build production

## Capabilities

### New Capabilities

- `vercel-production-exclude`: Cấu hình Vercel production build để exclude các dev-only packages (như `gitnexus`, `tree-sitter-cli`) không ảnh hưởng đến runtime

### Modified Capabilities

_(None)_

## Impact

- **File**: `pathora/package.json` — remove `gitnexus` từ dependencies
- **File**: `pathora/vercel.json` — thêm config để omit devDependencies
- **CI/CD**: Vercel build pipeline từ root `pathora/`
- **Risk thấp**: Không ảnh hưởng đến local dev (gitnexus vẫn hoạt động nếu cài local), không ảnh hưởng đến Next.js build output
