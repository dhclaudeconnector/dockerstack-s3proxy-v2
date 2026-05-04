# CHANGE LOGS (User-facing)

---

## [2.0.0] — 2026-04-09 — Modular Stack Template

### What's New

**Deploy any Docker image in minutes**
Change two lines in `.env` (`APP_IMAGE` and `APP_PORT`) and your app is live — no YAML editing required.

**Feature flags — enable what you need**
Turn ops tools on or off with simple env vars:

```env
ENABLE_DOZZLE=true
ENABLE_FILEBROWSER=true
ENABLE_WEBSSH=false
ENABLE_TAILSCALE=false
```

**Subdomains auto-generated**
Set `PROJECT_NAME=gitea` and `DOMAIN=example.com` once. All service URLs follow automatically:

- `gitea.example.com` → your app
- `logs.gitea.example.com` → log viewer
- `files.gitea.example.com` → file manager
- `ttyd.gitea.example.com` → web terminal

**One-command validation before deploy**

```bash
npm run dockerapp-validate:all
```

Checks env vars, Tailscale key format, and compose YAML — all at once.

**One-command deploy**

```bash
npm run dockerapp-exec:up
```

### What Changed (migration from v1)

If upgrading from the previous `docker-compose.yml` setup:

1. Replace `SUBDOMAIN_APP/DOZZLE/FILEBROWSER/WEBSSH` with just `PROJECT_NAME`
2. Replace `TAILSCALE_CLIENT_SECRET` with `TAILSCALE_AUTHKEY`
3. Replace `docker compose up` with `bash docker-compose/scripts/dc.sh up` or `npm run dockerapp-exec:up`
4. Update `cloudflared/config.yml` manually from `cloudflared/config.yml.example` if you use Cloudflare Tunnel

See `docs/DEPLOY.md` for the full migration guide.

---

## [2.0.1] — 2026-04-16 — Admin Tabs + File Manager

### Có gì mới

- Trang admin đã được tách thành các tab rõ ràng: Tổng quan, Cron jobs, Accounts, Public Bucket Proxy, Tất cả file, Runtime, Logs
- Có thêm tab **Tất cả file** để xem file của tất cả account hoặc lọc theo từng account
- Có thể **replace** hoặc **xóa** file đã track ngay trong admin mà không cần reload cả trang
- Cron jobs có thể **run trực tiếp** và xem log / báo cáo chi tiết ngay trên giao diện

### Cải thiện trải nghiệm dùng

- Bảng **Accounts** mặc định sắp theo ngày thêm account mới nhất
- Có thể bấm vào header để sắp xếp account theo các tiêu chí chính
- Tab **Public Bucket Proxy** chỉ tải danh sách file khi bạn bấm refresh, giúp vào trang nhanh hơn
- Các thao tác nặng đã có **loading**, **thông báo thành công / thất bại**, và refresh dữ liệu cục bộ theo từng tab
- Khi lưu account thành công, form account và phần thêm nhanh từ dịch vụ S3 sẽ tự clear

---

## 2026-04-16 - tăng tốc image trong compose.ops.yml
- Sửa image `webssh` để tránh bước `apt-get` trên Ubuntu vốn kéo package rất chậm.
- Chuyển `webssh` sang base Alpine + `openssh-client-default`, nên thời gian build lại ngắn hơn đáng kể.
- Pin version cho `dozzle` và `webssh-windows`, đồng thời thêm `pull_policy: missing` để hạn chế tự kéo lại image khi local đã có sẵn.

## 2026-04-17 - cron jobs dựng sẵn + API gọi từ cron bên ngoài
- Tab **Cron jobs** giờ luôn có sẵn 3 job: `probe_active_accounts`, `keepalive_touch`, `keepalive_scan`.
- Bạn có thể bấm **run now** ngay, không cần lưu cron job trước.
- Có thêm API protected: `POST /api/cron-jobs/:jobId/run` để bạn đặt cron ở máy / hệ thống khác rồi gọi vào.
- Nếu server này tắt mở liên tục và bạn đặt `CRON_ENABLED=false`, các job dựng sẵn vẫn hiện trong UI và vẫn chạy được bằng tay hoặc qua API.
## 2026-04-17
- Admin UI: thêm tab App access để mở nhanh domain / Tailscale / custom env URL cho S3Proxy, WebSSH, Dozzle, Filebrowser.
- Admin UI: thêm footer Runner info hiển thị 5 key chính và modal xem toàn bộ _DOTENVRTDB_RUNNER_*.
- Admin API /admin/api/overview: trả thêm runnerInfo và dockerAccess (inferred URLs + custom env URLs).
- compose.apps.yml / .env.example: bổ sung hướng dẫn pass _DOTENVRTDB_RUNNER_* và _DOCKER_ACCESS_URL_* vào container.


## 2026-05-04 - Thêm 5 logo Admin/PWA chọn nhanh bằng .env

### Có gì mới

- Đã thêm sẵn 5 bộ logo cho trang Admin/PWA.
- Mỗi bộ có đủ SVG, PNG nhiều kích cỡ và favicon.ico.
- Có file xem nhanh logo: `.docker-volumes/s3proxy-admin-assets/branding-preview.png`.
- Có file `.env.branding-presets` chứa sẵn 5 block cấu hình. Bạn chỉ cần copy/uncomment đúng một block vào `.env`, rồi restart stack.

### Cách dùng nhanh

1. Mở `branding-preview.png` để chọn logo.
2. Mở `.env.branding-presets` hoặc `.env.example`.
3. Bỏ comment đúng một block `LOGO OPTION` muốn dùng.
4. Restart stack bằng:

```bash
npm run dockerapp-exec:down
npm run dockerapp-exec:up
```

Nếu không bật block nào, app vẫn dùng logo và tên mặc định.

## 2026-05-04 - Sửa lỗi tạo app chỉ hiện chữ cái thay vì logo

### Đã sửa

- Khi cài Admin UI ra màn hình chính, icon không còn chỉ phụ thuộc vào SVG `/admin/icon` nữa.
- Manifest PWA giờ tự khai báo các icon PNG có sẵn như `512x512`, `384x384`, `192x192`, giúp Chrome/Samsung Browser nhận logo ổn định hơn.
- File preset `.env.branding-presets` và `.env.example` đã đổi khuyến nghị sang dùng `admin-logo-512.png`.
- Có thêm route `/admin/icons/...` và `/admin/apple-touch-icon.png` để trình duyệt lấy đúng file icon phù hợp.

### Lưu ý khi dùng

Nếu app/shortcut cũ đã được cài ra màn hình chính và đang hiện chữ cái, hãy gỡ app/shortcut đó rồi cài lại sau khi restart stack. Một số điện thoại giữ cache icon đã cài ở cấp hệ điều hành.

## 2026-05-04 - Bổ sung Admin PWA deeplink vào tab Accounts

### Đã thêm

- Thêm route `/admin/deeplink` để ứng dụng khác có thể mở Admin PWA bằng HTTPS deeplink.
- Khi nhận deeplink, UI tự mở tab **Accounts** và đưa dữ liệu URL vào textarea `accountServiceRawInput`.
- Hỗ trợ query key `raw`, `data`, `payload`, `text`, `value`, `accountServiceRawInput` và các biến thể base64url như `raw_b64`.
- Có toast thông báo nhận dữ liệu thành công.
- Service worker đã đổi cache version sang `s3proxy-admin-v5` và không cache URL deeplink có query data.

### Tài liệu

Xem hướng dẫn triển khai và kiểm thử tại `docs/services/admin-deeplink.md`.
