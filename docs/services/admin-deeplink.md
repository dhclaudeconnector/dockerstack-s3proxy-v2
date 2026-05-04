# Admin PWA Deep Link vào tab Accounts

Tài liệu này mô tả cơ chế deeplink mới cho Admin PWA. Tính năng này chỉ bổ sung luồng nhận dữ liệu từ URL vào giao diện hiện có, không tự parse, không tự lưu account và không thay đổi logic S3Proxy hiện tại.

## Mục tiêu

Khi một ứng dụng khác mở URL deeplink:

```text
https://<domain>/admin/deeplink?raw=<du_lieu_da_encode>
```

Admin PWA sẽ tự động:

1. Mở tab `Accounts` tương ứng nút `data-tab="accounts"`.
2. Đưa dữ liệu nhận từ URL vào textarea `id="accountServiceRawInput"`.
3. Hiện thông báo thành công: đã nhận dữ liệu deeplink.

## URL được hỗ trợ

### Cách khuyến nghị: dùng `raw` + `encodeURIComponent`

Ứng dụng gọi deeplink nên encode dữ liệu trước khi ghép vào URL:

```js
const raw = `supabase.com.database=...\nsupabase.com.accessToken=...\nemailOwner=...`
const url = `https://<domain>/admin/deeplink?raw=${encodeURIComponent(raw)}`
window.open(url, '_blank')
```

Ví dụ test nhanh:

```text
https://<domain>/admin/deeplink?raw=supabase.com.database%3Ddemo%0AemailOwner%3Dtest%40example.com
```

### Các query key dạng text được hỗ trợ

Có thể dùng một trong các key sau:

```text
raw
data
payload
text
value
accountServiceRawInput
```

Ví dụ:

```text
https://<domain>/admin/deeplink?data=<du_lieu_da_encode>
```

### Cách an toàn hơn với chuỗi dài/ký tự đặc biệt: base64url

Có thể truyền dữ liệu bằng base64url để tránh lỗi ký tự `&`, `=`, `+`, xuống dòng:

```text
https://<domain>/admin/deeplink?raw_b64=<base64url_payload>
```

Các key base64 được hỗ trợ:

```text
raw_b64
data_b64
payload_b64
accountServiceRawInput_b64
```

Ví dụ tạo base64url trong JavaScript:

```js
function toBase64Url(text) {
  const bytes = new TextEncoder().encode(text)
  let binary = ''
  bytes.forEach((byte) => { binary += String.fromCharCode(byte) })
  return btoa(binary).replace(/\+/g, '-').replace(/\//g, '_').replace(/=+$/g, '')
}

const raw = `supabase.com.database=...\nsupabase.com.accessToken=...\nemailOwner=...`
const url = `https://<domain>/admin/deeplink?raw_b64=${toBase64Url(raw)}`
window.open(url, '_blank')
```

## Hash fallback

Ngoài query string trực tiếp, app cũng hỗ trợ hash query dạng:

```text
https://<domain>/admin/deeplink#deeplink?raw=<du_lieu_da_encode>
```

hoặc:

```text
https://<domain>/admin/#deeplink?raw=<du_lieu_da_encode>
```

Dạng query trực tiếp `/admin/deeplink?raw=...` vẫn là cách nên dùng.

## Triển khai

Sau khi chép source mới vào server:

```bash
npm run dockerapp-exec:down
npm run dockerapp-exec:up
```

Hoặc nếu chỉ restart container app:

```bash
npm run dockerapp-exec:restart:app
```

Nếu PWA đã được cài trên điện thoại, nên đóng hẳn app rồi mở lại. Service worker đã đổi cache version sang `s3proxy-admin-v5` để trình duyệt lấy bản mới.

## Kiểm thử

### 1. Kiểm tra route trả HTML

Mở trình duyệt:

```text
https://<domain>/admin/deeplink?raw=test-deeplink
```

Kỳ vọng:

- Admin UI mở bình thường.
- Tab `Accounts` được active.
- Ô `accountServiceRawInput` có giá trị `test-deeplink`.
- Có toast thông báo nhận dữ liệu thành công.

### 2. Kiểm tra dữ liệu nhiều dòng

Mở URL:

```text
https://<domain>/admin/deeplink?raw=supabase.com.database%3Ddemo%0Asupabase.com.accessToken%3Dtoken-demo%0AemailOwner%3Dtest%40example.com
```

Kỳ vọng textarea nhận đúng 3 dòng:

```text
supabase.com.database=demo
supabase.com.accessToken=token-demo
emailOwner=test@example.com
```

### 3. Kiểm tra PWA trên điện thoại

1. Cài Admin PWA ra màn hình chính.
2. Từ app khác hoặc từ trình duyệt, mở link:

```text
https://<domain>/admin/deeplink?raw=test-from-mobile
```

Kỳ vọng:

- Nếu trình duyệt/PWA hỗ trợ mở link trong scope, link sẽ mở vào PWA.
- Nếu thiết bị mở bằng tab trình duyệt, logic vẫn hoạt động tương tự: mở Accounts và đưa dữ liệu vào textarea.

### 4. Kiểm tra không ảnh hưởng logic hiện tại

Mở các URL cũ:

```text
https://<domain>/admin
https://<domain>/admin/
```

Kỳ vọng:

- Admin UI vẫn mở tab mặc định `Tổng quan`.
- Không có dữ liệu nào tự động điền vào `accountServiceRawInput`.
- Các nút Refresh, Accounts, Cron jobs, Public Bucket Proxy vẫn hoạt động như trước.

## Ghi chú bảo mật

Dữ liệu truyền trên URL có thể xuất hiện trong browser history, log reverse proxy, log CDN hoặc screenshot. Nếu dữ liệu chứa token/secret nhạy cảm, nên ưu tiên một trong hai cách:

1. Dùng `raw_b64` để giảm lỗi ký tự, nhưng lưu ý base64 không phải mã hóa bảo mật.
2. Tốt hơn: app khác truyền một token ngắn hạn dùng một lần, rồi backend đổi token đó thành dữ liệu thật.

Trong bản này, service worker đã được chỉnh để không cache các URL deeplink hoặc URL `/admin` có query string, nhằm tránh lưu chuỗi raw vào Cache Storage của trình duyệt.
