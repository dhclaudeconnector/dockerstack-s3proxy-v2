# S3Proxy Admin branding assets

Thư mục này được mount vào container tại `/app/admin-assets`.

Bạn chỉ cần chọn một preset trong `.env.branding-presets` hoặc `.env.example`, bỏ comment đúng block đó, rồi restart stack.

## Logo options

- `01-s3-shield` — S3Proxy Shield: `/app/admin-assets/logo-options/01-s3-shield/admin-logo-512.png`
- `02-cloud-stack` — Cloud Stack Admin: `/app/admin-assets/logo-options/02-cloud-stack/admin-logo-512.png`
- `03-vault-key` — Vault Storage Admin: `/app/admin-assets/logo-options/03-vault-key/admin-logo-512.png`
- `04-edge-rocket` — Edge Rocket Admin: `/app/admin-assets/logo-options/04-edge-rocket/admin-logo-512.png`
- `05-orange-bucket` — Orange Bucket Admin: `/app/admin-assets/logo-options/05-orange-bucket/admin-logo-512.png`

## Kích cỡ đã tạo

Mỗi option có:

- `admin-logo-512.png`: khuyến nghị dùng cho `S3PROXY_ADMIN_ICON_PATH`, ổn định nhất khi cài PWA trên Android/Chrome/Samsung Browser.
- `admin-logo-192.png`: kích cỡ phổ biến cho Android/PWA.
- `admin-logo-180.png`: kích cỡ phổ biến cho Apple touch icon.
- `admin-logo.svg`: dùng tốt cho logo trong UI; code vẫn tự dò PNG cùng thư mục để khai báo manifest.
- `favicon.ico`: gồm nhiều size favicon 16/32/48/64/128/256.

Lưu ý: codebase hiện tự dò các file `admin-logo-*.png` nằm cùng thư mục với `S3PROXY_ADMIN_ICON_PATH` và khai báo đủ trong `/admin/manifest.webmanifest`. Vì vậy chỉ cần trỏ vào một file trong thư mục logo option, khuyến nghị là `admin-logo-512.png`.
