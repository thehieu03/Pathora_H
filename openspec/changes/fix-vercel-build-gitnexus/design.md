## Context

**Workspace structure:**
```
D:\Doan2\
├── pathora/              ← Vercel deploy root
│   ├── vercel.json       ← buildCommand: "npm install && npm run build"
│   ├── package.json      ← gitnexus@1.4.9 nằm trong dependencies(!)
│   └── frontend/         ← Next.js app (thực chất là app được build)
│       ├── vercel.json
│       └── package.json  ← gitnexus KHÔNG có ở đây
```

**Vấn đề kỹ thuật:**
- `gitnexus` là CLI tool với native addon (`tree-sitter` — C++ extension)
- Native addon cần compile trên target machine
- Vercel build machine: Node 24.14.1 + gcc/Linux
- V8 headers trong Node 24 yêu cầu C++20 (`#error "C++20 or later required."`)
- gcc trên Vercel không có đủ C++20 support
- → `node-gyp build` fail

**Tại sao local không lỗi:**
- Windows + MSVC có sẵn C++20 support
- Hoặc có pre-built binary cho Windows

## Goals / Non-Goals

**Goals:**
- Vercel build phải pass (exit code 0)
- `gitnexus` vẫn hoạt động trên local development nếu dev cần
- Không ảnh hưởng đến Next.js build output
- Thay đổi tối thiểu, low-risk

**Non-Goals:**
- Không cần fix gitnexus compile trên Vercel (không worth effort)
- Không cần thay đổi backend (panthora_be) — không liên quan
- Không cần tạo pre-built binary cho gitnexus

## Decisions

### Decision 1: Remove gitnexus from pathora/package.json dependencies

**Chọn:** Xóa `gitnexus` khỏi `pathora/package.json`

**Lý do:** `gitnexus` là tool cho local development (code intelligence, static analysis). Nó hoàn toàn không cần thiết khi:
- Deploy lên Vercel (chỉ chạy Next.js build + serve static)
- Hoạt động trên production

**Alternatives considered:**

| Option | Pros | Cons |
|--------|------|------|
| **A. Remove (chọn)** | Clean, zero footprint in prod | Devs cần install lại nếu cần |
| B. Move to devDependencies | Giữ declaration trong repo | gitnexus vẫn nằm trong npm tree, potential hoisting |
| C. .npmrc with omit=dev | Precise control | Extra config file, có thể ảnh hưởng local |

### Decision 2: Update vercel.json installCommand

**Chọn:** Thêm `--omit=dev` vào install step

**Lý do:** Defense-in-depth — đảm bảo rằng ngay cả khi ai đó thêm lại gitnexus vào devDependencies trong tương lai, nó vẫn không được install trên Vercel.

```
npm install --omit=dev
```

## Risks / Trade-offs

| Risk | Mitigation |
|------|------------|
| Dev muốn dùng gitnexus phải cài lại manual | Doc rõ trong commit message, gitnexus không cần thiết cho 99% dev workflow |
| gitnexus ở root package.json có thể là dependency của backend tooling | Kiểm tra panthora_be/ không phụ thuộc vào nó |

## Open Questions

- **Q1:** Tại sao `gitnexus` nằm ở root `pathora/package.json` mà không phải trong `panthora_be/`?
  - Có thể là copy-paste thừa từ backend sang, hoặc dev muốn dùng cho frontend analysis
  - **Resolution:** Remove — không cần cho frontend build
- **Q2:** Vercel deploy từ `pathora/` hay `pathora/frontend/`?
  - Build log show install chạy từ `pathora/` root
  - **Resolution:** Fix ở `pathora/package.json`
