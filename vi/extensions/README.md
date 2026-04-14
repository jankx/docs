# Hệ Thống Extension

Hệ Thống Extension của Jankx Theme cho phép bạn modularize chức năng và load tính năng động.

## Tổng Quan

```
┌─────────────────────────────────────────┐
│         ExtensionManager                │
│  ┌────────┐ ┌────────┐ ┌────────┐      │
│  │ Plugin │ │ Theme  │ │ Custom │      │
│  │ Ext A  │ │ Ext B  │ │ Ext C  │      │
│  └────┬───┘ └────┬───┘ └────┬───┘      │
│       └──────────┴──────────┘           │
│              Manifest                    │
└─────────────────────────────────────────┘
```

## Mục Lục

- [Khái Niệm Cốt Lõi](#khái-niệm-cốt-lõi)
- [Extension Manager](#extension-manager)
- [Tạo Extensions](#tạo-extensions)
- [Vòng Đời Extension](#vòng-đời-extension)

---

## Khái Niệm Cốt Lõi

### ExtensionManager

```php
use Jankx\Extensions\ExtensionManager;

// Get singleton instance
$manager = ExtensionManager::getInstance();

// Kiểm tra extension đang active
if ($manager->is_extension_active('my-extension')) {
    // Extension đang chạy
}

// Lấy tất cả extensions
$extensions = $manager->get_extensions();
```

### Các Loại Extension

| Loại | Mô Tả | Vị Trí |
|------|-------|--------|
| **Core** | Built-in extensions | `includes/extensions/` |
| **Theme** | Theme-specific | `includes/app/Extensions/` |
| **Plugin** | Third-party | Via plugin API |

---

## Extension Manager

### API Reference

```php
// Kích hoạt/Vô hiệu hóa
$manager->activate_extension('my-extension');
$manager->deactivate_extension('my-extension');

// Cài đặt/Gỡ cài đặt
$manager->install_extension('my-extension');
$manager->uninstall_extension('my-extension');

// Yêu cầu
$manager->require_extension('required-ext', true);   // Bắt buộc
$manager->require_extension('optional-ext', false);  // Tùy chọn

// Truy vấn
$manager->is_extension_active('my-extension');
$manager->is_extension_required('my-extension');
$manager->is_extension_installed('my-extension');
```

---

## Tạo Extensions

### Bước 1: Tạo Class Extension

```php
<?php
namespace Jankx\Extensions\MyExtension;

use Jankx\Extensions\AbstractExtension;

class MyExtension extends AbstractExtension
{
    protected $id = 'my-extension';
    protected $name = 'My Extension';
    protected $version = '1.0.0';
    protected $author = 'Your Name';
    protected $description = 'Extension description';
    
    public function activate(): void
    {
        parent::activate();
        $this->register_post_types();
    }
    
    public function boot(): void
    {
        add_action('init', [$this, 'init']);
    }
    
    private function register_post_types(): void
    {
        register_post_type('my_cpt', [
            'public' => true,
            'label' => 'My Custom Post Type',
        ]);
    }
}
```

### Bước 2: Tạo Manifest

```json
{
    "name": "My Extension",
    "id": "my-extension",
    "version": "1.0.0",
    "author": "Your Name",
    "description": "Extension description",
    "main": "MyExtension.php",
    "class": "Jankx\\Extensions\\MyExtension\\MyExtension",
    "requires": {
        "php": ">=7.4",
        "wordpress": ">=5.8"
    }
}
```

### Bước 3: Đăng Ký Extension

```php
add_action('jankx/extensions/init', function(ExtensionManager $manager) {
    $manager->add_extension('my-extension', new MyExtension());
});
```

---

## Vòng Đời Extension

```
┌─────────┐    ┌──────────┐    ┌─────────┐    ┌────────┐
│ Install │ -> │ Activate │ -> │   Boot  │ -> │  Init  │
└─────────┘    └──────────┘    └─────────┘    └────────┘
                                    │
                                    v
                             ┌──────────┐
                             │ Deactivate│
                             └──────────┘
```

### Các Phương Thức Khả Dụng

```php
class MyExtension extends AbstractExtension
{
    public function install(): bool
    {
        // Tạo tables, options
        return true;
    }
    
    public function activate(): void
    {
        // Đăng ký hooks
    }
    
    public function boot(): void
    {
        // Khởi tạo services
    }
    
    public function deactivate(): void
    {
        // Cleanup
    }
    
    public function uninstall(): bool
    {
        // Xóa tables, options
        return true;
    }
}
```

---

## Tính Năng Pro

### License Management ![Pro](https://img.shields.io/badge/PRO-764ba2?style=flat-square)

```php
use Jankx\Facades\License;

// Kiểm tra license
if (License::is_valid()) {
    // License hợp lệ
}

// Kích hoạt license
License::activate('your-license-key');

// Thông tin license
$info = License::get_info();
```

### Auto-Updates ![Pro](https://img.shields.io/badge/PRO-764ba2?style=flat-square)

```php
// Tự động cập nhật từ server
add_filter('jankx/extension/auto_update', function($enabled, $extension) {
    return $enabled;
}, 10, 2);
```

---

## Best Practices

1. **Namespace đúng cách**: Dùng `Jankx\Extensions\YourExtension`
2. **Dependency Injection**: Inject dependencies qua constructor
3. **Kiểm tra dependencies**: Kiểm tra extension phụ thuộc
4. **Version management**: Quản lý phiên bản cẩn thận
5. **Error handling**: Xử lý lỗi với try-catch
