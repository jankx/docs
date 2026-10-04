# Jankx Fast AJAX – Tài liệu kiến trúc & Hướng dẫn Migration

> **Mục tiêu:** Thay thế toàn bộ endpoint `admin-ajax.php` (chậm ~150–400ms) bằng hệ thống Fast AJAX mới dựa trên **WordPress SHORTINIT + Fat-Free Framework 3.9**, đạt tốc độ **< 10ms boot time**.

---

## Mục lục

1. [Tại sao cần migration?](#1-tại-sao-cần-migration)
2. [Kiến trúc tổng quan](#2-kiến-trúc-tổng-quan)
3. [Cấu trúc thư mục](#3-cấu-trúc-thư-mục)
4. [Luồng xử lý request](#4-luồng-xử-lý-request)
5. [URL Convention](#5-url-convention)
6. [Hướng dẫn tạo Controller mới](#6-hướng-dẫn-tạo-controller-mới)
7. [Middleware](#7-middleware)
8. [Response chuẩn](#8-response-chuẩn)
9. [Đăng ký namespace Extension](#9-đăng-ký-namespace-extension)
10. [Migration từ admin-ajax.php](#10-migration-từ-admin-ajaxphp)
11. [Mapping các endpoint cần migration](#11-mapping-các-endpoint-cần-migration)
12. [JavaScript Client](#12-javascript-client)
13. [Câu hỏi thường gặp](#13-câu-hỏi-thường-gặp)

---

## 1. Tại sao cần migration?

### Vấn đề của `admin-ajax.php`

| Vấn đề | Chi tiết |
|---|---|
| **Chậm** | WordPress boot đầy đủ: nạp tất cả plugins, theme, init hooks → 150–400ms |
| **Tốn tài nguyên** | Load WooCommerce, ACF, toàn bộ plugin dù request chỉ cần đọc 1 row DB |
| **Không có routing** | Mọi request đều gom vào 1 file, phân biệt nhau bằng `action` param |
| **Khó test** | Logic gắn chặt vào WP hooks, không tách biệt |
| **Không có middleware** | Phải viết lại auth/nonce/rate-limit cho từng action |

### Lợi ích của Jankx Fast AJAX

| Tiêu chí | `admin-ajax.php` | Jankx Fast AJAX |
|---|---|---|
| Boot time | ~150–400ms | **< 10ms** |
| WordPress context | Full boot | SHORTINIT (chỉ DB) |
| Routing | Flat action | **MVC + Pretty URL** |
| Middleware | Manual | **Pipeline tự động** |
| Namespace | 1 global | **Per-extension namespace** |
| Test | Khó | **Dễ (pure PHP class)** |
| Rate limiting | Không có | **Built-in** |

---

## 2. Kiến trúc tổng quan

```
┌─────────────────────────────────────────────────────────────┐
│                    Browser / API Client                      │
└──────────────────────────┬──────────────────────────────────┘
                           │  POST /jankx-ajax/my-account/profile/update
                           ▼
┌─────────────────────────────────────────────────────────────┐
│          Nginx / Apache (WordPress Rewrite Rules)           │
│   ^jankx-ajax/(.+)$ → index.php?jankx_fast_ajax=$1        │
└──────────────────────────┬──────────────────────────────────┘
                           │
             ┌─────────────┴─────────────┐
             │                           │
   ┌─────────▼──────────┐   ┌────────────▼────────────┐
   │  Via WordPress WP  │   │  Direct file access     │
   │  (template_redirect│   │  ajax.php               │
   │  → dispatchViaF3)  │   │  (SHORTINIT mode)       │
   └─────────┬──────────┘   └────────────┬────────────┘
             └─────────────┬─────────────┘
                           │
                           ▼
┌─────────────────────────────────────────────────────────────┐
│              Fat-Free Framework 3.9 Router (F3Router)       │
│                                                             │
│  Route: /jankx-ajax/@ns/@controller/@action[/@params]      │
│                                                             │
│  Namespace Registry:                                        │
│    "jankx"   → Jankx\Ajax\Controller\                      │
│    "ai"      → Jankx\Extensions\AiChatbox\Ajax\Controller\ │
│    "account" → Jankx\Extensions\MyAccount\Ajax\Controller\ │
│    ...       → ...                                          │
└──────────────────────────┬──────────────────────────────────┘
                           │
                           ▼
┌─────────────────────────────────────────────────────────────┐
│                   Middleware Pipeline                        │
│                                                             │
│  [NonceMiddleware] → [RateLimitMiddleware] → [AuthMiddleware]│
└──────────────────────────┬──────────────────────────────────┘
                           │
                           ▼
┌─────────────────────────────────────────────────────────────┐
│              Controller::action(array $params)              │
│                                                             │
│   - Đọc input()                                             │
│   - Gọi Service / $wpdb                                     │
│   - Trả về success() hoặc error()                           │
└──────────────────────────┬──────────────────────────────────┘
                           │
                           ▼
┌─────────────────────────────────────────────────────────────┐
│                    JsonResponse                             │
│  { "success": true, "data": {...}, "time_ms": 4.2 }        │
└─────────────────────────────────────────────────────────────┘
```

---

## 3. Cấu trúc thư mục

```
wp-content/themes/jankx/
│
├── ajax.php                                ← Entry point (SHORTINIT mode)
│
├── includes/framework/Ajax/
│   ├── routes.php                          ← Đăng ký namespace extensions
│   │
│   ├── Router/
│   │   └── F3Router.php                   ← Fat-Free Framework routing
│   │
│   ├── Controller/
│   │   ├── AbstractController.php         ← Base class – helpers, DB
│   │   └── PingController.php             ← Health-check endpoint
│   │
│   ├── Middleware/
│   │   ├── MiddlewareInterface.php        ← Contract
│   │   ├── NonceMiddleware.php            ← Xác thực WP nonce
│   │   └── RateLimitMiddleware.php        ← Giới hạn request/IP
│   │
│   └── Response/
│       └── JsonResponse.php               ← Chuẩn hóa + đo time_ms
│
└── includes/framework/Support/Providers/
    └── AjaxServiceProvider.php            ← WP integration (rewrite, nonce)
```

### Convention đặt tên Controller trong Extension

```
extensions/<extension-name>/
└── src/
    └── Ajax/
        └── Controller/
            ├── <Resource>Controller.php   ← Một controller = một resource
            └── ...
```

---

## 4. Luồng xử lý request

```
1. Client gửi: POST /jankx-ajax/account/profile/update
                    ↑ns    ↑controller  ↑action

2. WordPress rewrite → index.php?jankx_fast_ajax=account/profile/update
   HOẶC trực tiếp ajax.php (SHORTINIT)

3. F3Router::dispatch():
   - Tách ns="account", controller="profile", action="update"
   - Resolve class: Jankx\Extensions\MyAccount\Ajax\Controller\ProfileController

4. Middleware pipeline chạy tuần tự:
   - NonceMiddleware::handle()     → kiểm tra X-WP-Nonce header
   - RateLimitMiddleware::handle() → kiểm tra 60req/60s per IP

5. ProfileController::update(['extra', 'params'])

6. JsonResponse::success(['user' => ...]) → gửi JSON + thoát
```

---

## 5. URL Convention

### Pattern chính

```
/jankx-ajax/{namespace}/{controller}/{action}
/jankx-ajax/{namespace}/{controller}/{action}/{params...}
```

### Quy tắc đặt tên

| Thành phần | Format | Ví dụ |
|---|---|---|
| `namespace` | kebab-case (slug) | `account`, `tour-builder`, `ai` |
| `controller` | kebab-case → PascalCase + Controller | `profile` → `ProfileController` |
| `action` | kebab-case → camelCase | `update-avatar` → `updateAvatar()` |
| `params` | `/` phân cách, nhận dưới dạng array | `/123/detail` → `['123', 'detail']` |

### Ví dụ URL thực tế

| HTTP | URL | Controller::method |
|---|---|---|
| GET | `/jankx-ajax/jankx/ping/index` | `PingController::index()` |
| POST | `/jankx-ajax/account/profile/update` | `ProfileController::update()` |
| POST | `/jankx-ajax/account/profile/upload-avatar` | `ProfileController::uploadAvatar()` |
| POST | `/jankx-ajax/account/verification/send-email` | `VerificationController::sendEmail()` |
| GET | `/jankx-ajax/metrics/post-view/track/123` | `PostViewController::track(['123'])` |
| POST | `/jankx-ajax/tour/builder/submit` | `BuilderController::submit()` |
| POST | `/jankx-ajax/comment/media/upload` | `MediaController::upload()` |
| POST | `/jankx-ajax/ai/chat/send` | `ChatController::send()` |

---

## 6. Hướng dẫn tạo Controller mới

### Bước 1: Tạo file Controller

```php
<?php
// extensions/my-account/src/Ajax/Controller/ProfileController.php

namespace Jankx\Extensions\MyAccount\Ajax\Controller;

use Jankx\Ajax\Controller\AbstractController;
use Jankx\Ajax\Middleware\NonceMiddleware;
use Jankx\Ajax\Middleware\RateLimitMiddleware;

class ProfileController extends AbstractController
{
    // Middleware chạy trước mọi action trong controller này
    protected array $middlewares = [
        NonceMiddleware::class,
        RateLimitMiddleware::class,
    ];

    /**
     * POST /jankx-ajax/account/profile/update
     */
    public function update(array $params = []): void
    {
        $name  = sanitize_text_field($this->input('display_name', ''));
        $email = sanitize_email($this->input('email', ''));

        if (empty($name)) {
            $this->error('Tên không được để trống.', 422);
            return;
        }

        $updated = $this->db()->update(
            $this->db()->users,
            ['display_name' => $name, 'user_email' => $email],
            ['ID' => get_current_user_id()]
        );

        if ($updated === false) {
            $this->error('Cập nhật thất bại.', 500);
            return;
        }

        $this->success(['message' => 'Cập nhật thành công.']);
    }

    /**
     * POST /jankx-ajax/account/profile/upload-avatar
     */
    public function uploadAvatar(array $params = []): void
    {
        // Xử lý file upload
        $this->success(['avatar_url' => 'https://...']);
    }
}
```

### Bước 2: Đăng ký namespace bằng manifest.json

Thay vì sửa file core của Jankx, bạn mở file `manifest.json` của extension và khai báo:

```json
{
    ...
    "ajax_slug": "account",
    "ajax_namespace": "Jankx\\Extensions\\MyAccount\\Ajax\\Controller\\"
}
```

### Bước 3: Test endpoint

```bash
# Test ping (không cần nonce)
curl https://nibitour.vn/jankx-ajax/jankx/ping/index

# Test với nonce
NONCE=$(wp eval "echo wp_create_nonce('jankx_ajax');")
curl -X POST https://nibitour.vn/jankx-ajax/account/profile/update \
  -H "X-WP-Nonce: $NONCE" \
  -H "Content-Type: application/json" \
  -d '{"display_name": "Nguyen Van A"}'
```

---

## 7. Middleware

### Các Middleware có sẵn

| Class | Mô tả | Yêu cầu client |
|---|---|---|
| `NonceMiddleware` | Xác thực WP nonce | Header `X-WP-Nonce` hoặc `_wpnonce` |
| `RateLimitMiddleware` | Giới hạn 60 req/60s per IP | Không |

### Tạo Middleware tùy chỉnh

```php
<?php
namespace Jankx\Ajax\Middleware;

use Base;
use Jankx\Ajax\Response\JsonResponse;

class AuthMiddleware implements MiddlewareInterface
{
    public function handle(Base $f3): bool
    {
        // SHORTINIT không tự load pluggable.php, cần require thêm
        if (! function_exists('is_user_logged_in')) {
            require_once ABSPATH . 'wp-includes/pluggable.php';
        }

        if (! is_user_logged_in()) {
            JsonResponse::error('Ban can dang nhap.', 401)->send();
            return false;
        }

        return true;
    }
}
```

### Gắn middleware vào Controller

```php
class ProfileController extends AbstractController
{
    protected array $middlewares = [
        NonceMiddleware::class,     // Chạy thứ 1
        AuthMiddleware::class,      // Chạy thứ 2
        RateLimitMiddleware::class, // Chạy thứ 3
    ];
}
```

---

## 8. Response chuẩn

### Format thành công

```json
{
    "success": true,
    "data": {
        "message": "Cap nhat thanh cong.",
        "user_id": 42
    },
    "time_ms": 4.2
}
```

### Format lỗi

```json
{
    "success": false,
    "error": "Ban can dang nhap.",
    "code": 401
}
```

### Dùng trong Controller

```php
// Thành công
$this->success(['key' => 'value']);
$this->success(['key' => 'value'], 201); // custom HTTP status code

// Lỗi
$this->error('Khong tim thay.', 404);
$this->error('Du lieu khong hop le.', 422, ['field' => 'email']);
```

---

## 9. Đăng ký namespace Extension

Hệ thống Fast AJAX tự động quét và nạp namespace từ file `manifest.json` của các extension (cả ở theme cha `jankx` và child theme).

Thay vì phải hardcode khai báo trong file `routes.php` như trước, mỗi extension chỉ cần thêm 2 field `ajax_slug` và `ajax_namespace` vào `manifest.json`.

### Cấu hình `manifest.json`

```json
{
    "name": "Tên Extension",
    "extension_id": "ten-extension",
    ...
    "ajax_slug": "slug-tren-url",
    "ajax_namespace": "Jankx\\Extensions\\TenExtension\\Ajax\\Controller\\"
}
```

- `ajax_slug`: Là thành phần `{namespace}` trên URL (`/jankx-ajax/{namespace}/...`)
- `ajax_namespace`: Là thư mục chứa các class Controller (nhớ dùng escape backslash `\\`).

### Ví dụ
Với khai báo:
```json
{
    "ajax_slug": "ecommerce",
    "ajax_namespace": "Jankx\\Extensions\\Ecommerce\\Ajax\\Controller\\"
}
```
URL `/jankx-ajax/ecommerce/cart/get` sẽ tự động route tới `Jankx\Extensions\Ecommerce\Ajax\Controller\CartController::get()`.

---

## 10. Migration từ admin-ajax.php

### Checklist cho mỗi endpoint

- [ ] Tạo `Controller` class trong `src/Ajax/Controller/`
- [ ] Move logic từ handler method cũ vào action method mới
- [ ] Thêm middleware phù hợp (Nonce, Auth, RateLimit)
- [ ] Khai báo `ajax_slug` và `ajax_namespace` trong `manifest.json` của extension
- [ ] Cập nhật JS: đổi `ajaxUrl` + `action` sang URL mới
- [ ] Test endpoint mới hoạt động (curl hoặc browser)
- [ ] Giữ lại endpoint cũ song song trong thời gian chuyển đổi
- [ ] Xóa `add_action('wp_ajax_*', ...)` cũ sau khi confirm

### Pattern migration

**Trước (admin-ajax.php):**

```php
// PHP – Handler cũ
class ProfileHandler
{
    public function register(): void
    {
        add_action('wp_ajax_jankx_update_profile', [$this, 'ajaxUpdateProfile']);
    }

    public function ajaxUpdateProfile(): void
    {
        check_ajax_referer('jankx_profile_nonce');
        $name = sanitize_text_field($_POST['display_name'] ?? '');
        // ... logic ...
        wp_send_json_success(['message' => 'OK']);
    }
}
```

```javascript
// JS cũ
jQuery.post(ajaxUrl, {
    action: 'jankx_update_profile',
    _wpnonce: nonce,
    display_name: name
}).done(res => {
    if (res.success) { /* ... */ }
});
```

**Sau (Jankx Fast AJAX):**

```php
// PHP – Controller mới
namespace Jankx\Extensions\MyAccount\Ajax\Controller;

use Jankx\Ajax\Controller\AbstractController;
use Jankx\Ajax\Middleware\NonceMiddleware;

class ProfileController extends AbstractController
{
    protected array $middlewares = [NonceMiddleware::class];

    public function update(array $params = []): void
    {
        $name = sanitize_text_field($this->input('display_name', ''));
        // ... logic giống hệt ...
        $this->success(['message' => 'OK']);
    }
}
```

```javascript
// JS mới
const res = await fetch(JankxAjax.url + '/account/profile/update', {
    method: 'POST',
    headers: {
        'Content-Type': 'application/json',
        'X-WP-Nonce':   JankxAjax.nonce,
    },
    body: JSON.stringify({ display_name: name }),
});
const data = await res.json();
// data = { success: true, data: { message: "OK" }, time_ms: 4.2 }
```

---

## 11. Mapping các endpoint cần migration

### Extension: `my-account`

| Endpoint cũ (action) | URL mới | Controller::action | Middleware |
|---|---|---|---|
| `jankx_update_profile` | `POST /account/profile/update` | `ProfileController::update` | Nonce + Auth |
| `jankx_upload_avatar` | `POST /account/profile/upload-avatar` | `ProfileController::uploadAvatar` | Nonce + Auth |
| `jankx_change_password` | `POST /account/profile/change-password` | `ProfileController::changePassword` | Nonce + Auth |
| `jankx_save_settings` | `POST /account/profile/save-settings` | `ProfileController::saveSettings` | Nonce + Auth |
| `jankx_send_verify_email` | `POST /account/verification/send-email` | `VerificationController::sendEmail` | Nonce |

### Extension: `metrics`

| Endpoint cũ (action) | URL mới | Controller::action | Middleware |
|---|---|---|---|
| `track_post_view` | `POST /metrics/post-view/track/{id}` | `PostViewController::track` | RateLimit |
| `get_post_views` | `GET /metrics/post-view/get/{id}` | `PostViewController::get` | Không |

### Extension: `comment-media`

| Endpoint cũ (action) | URL mới | Controller::action | Middleware |
|---|---|---|---|
| `comment_media_upload` | `POST /comment/media/upload` | `MediaController::upload` | Nonce |

### Extension: `tour-builder`

| Endpoint cũ (action) | URL mới | Controller::action | Middleware |
|---|---|---|---|
| `jankx_tour_builder_tours` | `GET /tour/builder/list` | `BuilderController::list` | Nonce |
| `jankx_tour_builder_submit` | `POST /tour/builder/submit` | `BuilderController::submit` | Nonce |

### Extension: `coupon-system`

| Endpoint cũ (action) | URL mới | Controller::action | Middleware |
|---|---|---|---|
| `jankx_coupon_feedback` | `POST /coupon/feedback/submit` | `FeedbackController::submit` | Nonce + RateLimit |

### Extension: `ecommerce-product`

| Endpoint cũ (action) | URL mới | Controller::action | Middleware |
|---|---|---|---|
| `jankx_toggle_featured` | `POST /product/featured/toggle` | `FeaturedController::toggle` | Nonce + Auth(admin) |

### Extension: `ai-chatbox` (migrate từ REST API)

| Endpoint cũ (REST) | URL mới | Controller::action | Middleware |
|---|---|---|---|
| `POST /jankx/v1/ai-chat` | `POST /ai/chat/send` | `ChatController::send` | Nonce + RateLimit |
| `GET /jankx/v1/ai-chat/suggestions` | `GET /ai/chat/suggestions` | `ChatController::suggestions` | Không |
| `GET /jankx/v1/ai-chat/filter-options` | `GET /ai/chat/filter-options` | `ChatController::filterOptions` | Không |

---

## 12. JavaScript Client

`window.JankxAjax` được inject tự động bởi `AjaxServiceProvider` vào mọi trang:

```javascript
window.JankxAjax = {
    url:   "https://nibitour.vn/jankx-ajax",  // base URL
    nonce: "abc123xyz",                         // WP nonce (action: jankx_ajax)
    mode:  "rewrite"                            // "rewrite" | "direct"
};
```

### Helper function gợi ý

```javascript
/**
 * Jankx Fast AJAX client helper
 * Sử dụng: const data = await jankxAjax('account/profile/update', { name: 'A' })
 */
async function jankxAjax(path, body = {}, method = 'POST') {
    const url = window.JankxAjax.url + '/' + path;
    const opts = {
        method,
        headers: {
            'Content-Type': 'application/json',
            'X-WP-Nonce':   window.JankxAjax.nonce,
        },
    };
    if (method !== 'GET') {
        opts.body = JSON.stringify(body);
    }
    const res  = await fetch(url, opts);
    const data = await res.json();
    if (! data.success) throw new Error(data.error || 'Request failed');
    return data;
}
```

---

## 13. Câu hỏi thường gặp

**Q: SHORTINIT có đủ functions để dùng không?**

SHORTINIT nạp: `$wpdb`, `get_option()`, `wp_verify_nonce()`, `sanitize_*()`, `esc_*()`.

Không có sẵn (cần require thêm):
- `is_user_logged_in()` → `require ABSPATH . 'wp-includes/pluggable.php'`
- `wp_handle_upload()` → `require ABSPATH . 'wp-admin/includes/file.php'`
- `WP_Query` → **Không dùng, truy vấn thẳng qua `$wpdb`**

**Q: Khi nào vẫn nên dùng REST API / admin-ajax?**

- Upload file cần `media_handle_upload()` với nhiều context
- Endpoint cần WooCommerce cart, ACF field API
- Endpoint của third-party plugin không thể migrate

**Q: Làm sao test performance?**

```bash
# Fast AJAX (mục tiêu < 10ms)
time curl -s https://nibitour.vn/jankx-ajax/jankx/ping/index

# admin-ajax.php (baseline ~150ms+)
time curl -s -X POST https://nibitour.vn/wp-admin/admin-ajax.php -d "action=heartbeat"
```

**Q: Rewrite rules không hoạt động?**

```bash
wp rewrite flush --hard
```
Hoặc: **Admin → Settings → Permalinks → Save Changes**.

---

## Changelog

| Phiên bản | Ngày | Thay đổi |
|---|---|---|
| 1.0.0 | 2026-10-05 | Khởi tạo hệ thống Fast AJAX với SHORTINIT + F3 Framework 3.9 |
