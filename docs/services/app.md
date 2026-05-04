# App service (`compose.apps.yml`) — S3 Proxy runtime

## Vai trò
- Service ứng dụng chính, build từ `services/s3proxy`.
- Cung cấp S3-compatible proxy + metadata/runtime metrics.

## Cấu hình chính
- Image local tag: `${PROJECT_NAME}-s3proxy:local`
- Build context: `./services/s3proxy`
- Port expose localhost: `127.0.0.1:${APP_HOST_PORT}:${APP_PORT}`
- Data volume: `${DOCKER_VOLUMES_ROOT:-./.docker-volumes}/s3proxy-data:/app/.docker-volumes/s3proxy-data`
- Healthcheck: `wget http://localhost:${APP_PORT}${HEALTH_PATH}`

## ENV bắt buộc
- `APP_PORT`: port proxy lắng nghe trong container.
- `S3PROXY_API_KEY`, `S3PROXY_FIREBASE_RTDB_URL`, `S3PROXY_FIREBASE_DB_SECRET`: bắt buộc cho nghiệp vụ s3proxy.
- `PROJECT_NAME`, `DOMAIN`: tạo hostname public.
- `CADDY_AUTH_USER`, `CADDY_AUTH_HASH`: basic auth.

## ENV optional
- `APP_HOST_PORT` (default 3000): truy cập localhost host machine.
- `NODE_ENV` (default production).
- `HEALTH_PATH` (default `/health`).
- `DOCKER_VOLUMES_ROOT` (default `./.docker-volumes`).
- `S3PROXY_SQLITE_PATH` (default `./.docker-volumes/s3proxy-data/routes.db`).
- `S3PROXY_DEPLOY_VERSION` (default tự sinh UTC format `YYYY-MM-DD: HH:mm UTC`): hiển thị version deploy trong admin UI.
- `S3PROXY_CRON_ENABLED`, `S3PROXY_CRON_TIMEZONE`, `S3PROXY_CRON_RUN_ON_START`.
- `S3PROXY_CRON_KEEPALIVE_ENABLED`, `S3PROXY_CRON_KEEPALIVE_EXPRESSION`, `S3PROXY_CRON_KEEPALIVE_MODE` (`scan` hoặc `touch`).
- `S3PROXY_CRON_KEEPALIVE_PREFIX`, `S3PROXY_CRON_KEEPALIVE_CONTENT_PREFIX`.
- `S3PROXY_ADMIN_TEST_PREFIX`: prefix object test đọc/ghi/xoá từ admin UI.
- `TAILSCALE_TAILNET_DOMAIN`: route HTTPS nội bộ qua caddy_1.

## Routing
- Public host: `${PROJECT_NAME}.${DOMAIN}` (+ alias).
- Internal HTTPS host: `${PROJECT_NAME}.${TAILSCALE_TAILNET_DOMAIN}` với `tls internal`.
- Admin UI: `GET /admin` (nên đặt sau Caddy Basic Auth nếu mở internet).
- Admin API: `GET /admin/api/overview`, `POST /admin/api/test-s3`.
- Cron API: `POST /admin/api/cron-jobs`, `POST /admin/api/cron-jobs/:jobId/run`, `DELETE /admin/api/cron-jobs/:jobId`.
- Trong UI `/admin` đã có form để tạo/sửa cron job runtime (keepalive/probe) và chạy thủ công từng job.

## Đổi logo, tên Admin UI và PWA qua `.env`

Các biến branding đều có mặc định; nếu không khai báo thì `/admin` vẫn dùng `S3Proxy Admin` và icon SVG mặc định trong source.

- `S3PROXY_ADMIN_APP_NAME`: tên đầy đủ hiển thị ở tab trình duyệt, header `/admin` và manifest PWA.
- `S3PROXY_ADMIN_APP_SHORT_NAME`: tên ngắn khi cài app trên màn hình chính.
- `S3PROXY_ADMIN_APP_DESCRIPTION`: mô tả app trong manifest.
- `S3PROXY_ADMIN_THEME_COLOR`: màu theme trình duyệt/mobile PWA, nên dùng HEX, ví dụ `#0b1020`.
- `S3PROXY_ADMIN_BACKGROUND_COLOR`: màu nền lúc PWA khởi động.
- `S3PROXY_ADMIN_ICON_PATH`: đường dẫn file icon bên trong container. Khuyến nghị đặt file ở `./.docker-volumes/s3proxy-admin-assets/` trên host và trỏ tới `/app/admin-assets/<ten-file>`.
- `S3PROXY_ADMIN_ICON_MIME`: MIME type của icon, ví dụ `image/svg+xml`, `image/png`, `image/x-icon`. Nếu để trống, app tự suy đoán theo đuôi file.
- `S3PROXY_ADMIN_ICON_SIZES`: kích cỡ khai báo trong manifest. SVG dùng `any`; PNG 512 dùng `512x512`; ICO có thể dùng `16x16 32x32 48x48 256x256`.
- `S3PROXY_ADMIN_ICON_PURPOSE`: mặc định `any maskable` để phù hợp PWA.

Ví dụ dùng PNG 512x512:

```env
S3PROXY_ADMIN_APP_NAME="My Storage Admin"
S3PROXY_ADMIN_APP_SHORT_NAME=Storage
S3PROXY_ADMIN_THEME_COLOR="#111827"
S3PROXY_ADMIN_BACKGROUND_COLOR="#111827"
S3PROXY_ADMIN_ICON_PATH=/app/admin-assets/logo-512.png
S3PROXY_ADMIN_ICON_MIME=image/png
S3PROXY_ADMIN_ICON_SIZES=512x512
S3PROXY_ADMIN_ICON_PURPOSE="any maskable"
```

Sau khi đổi `.env`, rebuild/recreate container để compose nạp env mới:

```bash
npm run dockerapp-exec:down
npm run dockerapp-exec:up
```

## Bộ logo/icon dựng sẵn cho Admin UI/PWA

Dự án đã có sẵn 5 bộ logo trong thư mục:

```text
./.docker-volumes/s3proxy-admin-assets/logo-options/
```

Khi chạy bằng compose, thư mục này được mount vào container tại:

```text
/app/admin-assets/logo-options/
```

Có thể xem nhanh 5 lựa chọn bằng file:

```text
./.docker-volumes/s3proxy-admin-assets/branding-preview.png
./.docker-volumes/s3proxy-admin-assets/branding-preview.html
```

Mỗi bộ logo có đủ các file thường dùng:

- `admin-logo.svg`: khuyến nghị dùng, nhẹ, nét, hợp PWA; cấu hình `image/svg+xml`, `sizes=any`.
- `admin-logo-512.png`: PNG 512x512, dùng tốt cho PWA manifest; cấu hình `image/png`, `sizes=512x512`.
- `admin-logo-192.png`: PNG 192x192 cho Android/PWA.
- `admin-logo-180.png`: PNG 180x180 cho Apple touch icon.
- `favicon.ico`: favicon nhiều kích cỡ 16/32/48/64/128/256.

Các preset env đã được viết sẵn trong `.env.example` và `.env.branding-presets`. Cách dùng nhanh là copy/uncomment đúng một block `LOGO OPTION`, sau đó restart stack.

Danh sách preset:

1. `01-s3-shield` — logo khiên bảo mật, hợp giao diện admin/proxy.
2. `02-cloud-stack` — logo cloud + stack, hợp triển khai cloud/storage.
3. `03-vault-key` — logo khóa/vault, hợp nhấn mạnh private/on-prem/security.
4. `04-edge-rocket` — logo rocket, hợp nhấn mạnh tốc độ/edge/proxy.
5. `05-orange-bucket` — logo bucket màu cam, hợp nhận diện S3/bucket rõ ràng.

Ví dụ dùng option 1 bằng SVG:

```env
S3PROXY_ADMIN_APP_NAME="S3Proxy Shield"
S3PROXY_ADMIN_APP_SHORT_NAME=S3Shield
S3PROXY_ADMIN_APP_DESCRIPTION="Secure S3 proxy admin console"
S3PROXY_ADMIN_THEME_COLOR="#0b1020"
S3PROXY_ADMIN_BACKGROUND_COLOR="#0b1020"
S3PROXY_ADMIN_ICON_PATH=/app/admin-assets/logo-options/01-s3-shield/admin-logo.svg
S3PROXY_ADMIN_ICON_MIME=image/svg+xml
S3PROXY_ADMIN_ICON_SIZES=any
S3PROXY_ADMIN_ICON_PURPOSE="any maskable"
```

