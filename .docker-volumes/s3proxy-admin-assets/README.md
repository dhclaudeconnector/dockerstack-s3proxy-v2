# S3Proxy Admin branding assets

Thư mục này được mount vào container tại `/app/admin-assets`.

Bạn chỉ cần chọn một preset trong `.env.branding-presets` hoặc `.env.example`, bỏ comment đúng block đó, rồi restart stack.

## Logo options

- `01-s3-shield` — S3Proxy Shield: `/app/admin-assets/logo-options/01-s3-shield/admin-logo.svg`
- `02-cloud-stack` — Cloud Stack Admin: `/app/admin-assets/logo-options/02-cloud-stack/admin-logo.svg`
- `03-vault-key` — Vault Storage Admin: `/app/admin-assets/logo-options/03-vault-key/admin-logo.svg`
- `04-edge-rocket` — Edge Rocket Admin: `/app/admin-assets/logo-options/04-edge-rocket/admin-logo.svg`
- `05-orange-bucket` — Orange Bucket Admin: `/app/admin-assets/logo-options/05-orange-bucket/admin-logo.svg`

## Kích cỡ đã tạo

Mỗi option có:

- `admin-logo.svg`: khuyến nghị dùng cho PWA vì nét, nhẹ, không vỡ hình.
- `admin-logo-512.png`: dùng tốt cho PWA manifest nếu muốn PNG.
- `admin-logo-192.png`: kích cỡ phổ biến cho Android/PWA.
- `admin-logo-180.png`: kích cỡ phổ biến cho Apple touch icon.
- `favicon.ico`: gồm nhiều size favicon 16/32/48/64/128/256.

Lưu ý: codebase hiện dùng một biến `S3PROXY_ADMIN_ICON_PATH` cho `/admin/icon`, manifest, favicon và apple-touch-icon; vì vậy preset khuyến nghị dùng SVG hoặc PNG 512x512.
