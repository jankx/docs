# Hệ Thống Quản Trị

Hệ Thống Quản Trị của Jankx Theme cung cấp công cụ toàn diện cho quản lý theme, xử lý form và theme options.

## Tổng Quan

```
┌─────────────────────────────────────────┐
│           Admin Panel                    │
│  ┌────────┐ ┌────────┐ ┌────────┐     │
│  │ Theme  │ │ Form   │ │ Admin  │     │
│  │Options │ │Handlers│ │Notices │     │
│  └────────┘ └────────┘ └────────┘     │
│         │           │          │        │
│         └───────────┴──────────┘        │
│              FormHandler                 │
└─────────────────────────────────────────┘
```

## Mục Lục

- [Form Handler](#form-handler)
- [Admin Notices](#admin-notices)
- [Theme Options](#theme-options)
- [Bảo Mật](#bảo-mật)

---

## Form Handler

Class `FormHandler` xử lý form submissions với kiểm tra bảo mật tích hợp.

### Sử Dụng

```php
use Jankx\Admin\Handlers\FormHandler;
use Jankx\Foundation\Application;

$app = Application::getInstance();
$formHandler = new FormHandler($app);

// Xử lý requests (thường gọi trên admin_init)
add_action('admin_init', [$formHandler, 'handleRequests']);
```

### Các Actions Có Sẵn

| Action | Mô Tả | Quyền |
|--------|-------|-------|
| `save_image_sizes` | Lưu image size settings | `manage_options` |
| `clear_debug_log` | Xóa debug log | `manage_options` |
| `activate_license` | Kích hoạt license | `manage_options` ![Pro](https://img.shields.io/badge/PRO-764ba2?style=flat-square) |
| `deactivate_license` | Vô hiệu hóa license | `manage_options` ![Pro](https://img.shields.io/badge/PRO-764ba2?style=flat-square) |

### Custom Actions

```php
add_action('jankx/form/handle/{action}', function($data, $formHandler) {
    update_option('my_custom_option', $data['value']);
    $formHandler->addAdminNotice('Đã lưu thành công!', 'success');
}, 10, 2);
```

---

## Admin Notices

Hiển thị thông báo cho admin users.

### Các Phương Thức

```php
// Thêm notice
$formHandler->addAdminNotice('Đã lưu!', 'success');
$formHandler->addAdminNotice('Lỗi xảy ra', 'error');
$formHandler->addAdminNotice('Cảnh báo', 'warning');
$formHandler->addAdminNotice('Thông tin', 'info');
```

### Các Loại Notice

| Loại | Màu | Trường Hợp |
|------|-----|------------|
| `success` | Xanh lá | Thành công |
| `warning` | Vàng | Cần chú ý |
| `error` | Đỏ | Thất bại |
| `info` | Xanh dương | Thông tin |

---

## Theme Options

### Service Provider

```php
namespace App\Providers;

use Jankx\Foundation\Providers\ServiceProvider;
use App\Services\ThemeOptionsService;

class ThemeOptionsServiceProvider extends ServiceProvider
{
    public function register(): void
    {
        $this->app->singleton('theme-options', function($app) {
            return new ThemeOptionsService($app);
        });
    }
    
    public function boot(): void
    {
        add_action('after_setup_theme', [$this->app->get('theme-options'), 'init']);
    }
}
```

### Options Service

```php
class ThemeOptionsService
{
    public function get(string $key, $default = null)
    {
        return $this->options[$key] ?? $default;
    }
    
    public function set(string $key, $value): void
    {
        $this->options[$key] = $value;
        update_option('jankx_options', $this->options);
    }
}
```

### Font Manager ![Pro](https://img.shields.io/badge/PRO-764ba2?style=flat-square)

```php
use Jankx\Facades\Fonts;

// Thêm font tùy chỉnh
Fonts::add([
    'name' => 'Custom Font',
    'family' => 'Custom, sans-serif',
    'category' => 'custom',
]);

// Lấy active fonts
$fonts = Fonts::getActive();
```

### Icon Manager ![Pro](https://img.shields.io/badge/PRO-764ba2?style=flat-square)

```php
use Jankx\Facades\Icons;

// Lấy icons theo type
$icons = Icons::get('fontawesome');
```

---

## Bảo Mật

### Nonce Verification

```php
// Tạo nonce
$nonce = wp_create_nonce('my_action');

// Verify trong handler
if (!wp_verify_nonce($_POST['_wpnonce'] ?? '', 'my_action')) {
    wp_die('Security check failed');
}
```

### Capability Checks

```php
// Kiểm tra quyền
if (!current_user_can('manage_options')) {
    wp_die('Bạn không có quyền');
}
```

### Data Sanitization

```php
// FormHandler cung cấp sanitization tích hợp
$data = $this->sanitizeRequestData($_POST);

// Manual sanitization
$clean = [
    'text' => sanitize_text_field($input['text']),
    'email' => sanitize_email($input['email']),
    'url' => esc_url_raw($input['url']),
    'html' => wp_kses_post($input['html']),
];
```

---

## Best Practices

1. **Luôn verify nonces**
2. **Kiểm tra capabilities**
3. **Validate input**
4. **Escape output**
5. **Dùng Admin Notices đúng cách**
