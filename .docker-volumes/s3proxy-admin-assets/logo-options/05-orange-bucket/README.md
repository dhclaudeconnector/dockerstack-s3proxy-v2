# Orange Bucket Admin

Ready-to-use icon assets for S3Proxy Admin/PWA branding.

Recommended env path for installed PWA app icon:

```env
S3PROXY_ADMIN_ICON_PATH=/app/admin-assets/logo-options/05-orange-bucket/admin-logo-512.png
S3PROXY_ADMIN_ICON_MIME=image/png
S3PROXY_ADMIN_ICON_SIZES=512x512
S3PROXY_ADMIN_ICON_PURPOSE="any maskable"
```

SVG alternative for the in-page UI logo. The code will still auto-detect the sibling PNG files for the PWA manifest when they are present in the same folder:

```env
S3PROXY_ADMIN_ICON_PATH=/app/admin-assets/logo-options/05-orange-bucket/admin-logo.svg
S3PROXY_ADMIN_ICON_MIME=image/svg+xml
S3PROXY_ADMIN_ICON_SIZES=any
S3PROXY_ADMIN_ICON_PURPOSE="any maskable"
```

Available files: SVG, PNG 512/384/192/180/128/64/48/32/16, and favicon.ico.
