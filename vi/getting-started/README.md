# Bắt Đầu với Jankx Theme

Chào mừng đến với Jankx Theme! Hướng dẫn này sẽ giúp bạn cài đặt và chạy theme một cách nhanh chóng.

## Mục Lục

- [Cài Đặt](#cài-đặt)
- [Yêu Cầu](#yêu-cầu)
- [Cấu Hình](#cấu-hình)
- [Các Bước Đầu Tiên](#các-bước-đầu-tiên)

---

## Cài Đặt

### Cách 1: Qua Composer (Khuyến Nghị)

```bash
composer require jankx/theme
```

### Cách 2: Cài Đặt Thủ Công

1. Tải bản release mới nhất từ [GitHub Releases](https://github.com/jankx/theme/releases)
2. Giải nén vào `/wp-content/themes/jankx/`
3. Kích hoạt trong WordPress Admin → Giao Diện → Themes

### Cách 3: Git Clone

```bash
cd /wp-content/themes/
git clone https://github.com/jankx/theme.git jankx
```

---

## Yêu Cầu

| Yêu Cầu | Phiên Bản |
|---------|-----------|
| PHP | >= 7.4 |
| WordPress | >= 5.8 |
| Composer | >= 2.0 |
| Node.js | >= 16.0 (cho development) |

---

## Cấu Hình

### Thiết Lập Môi Trường

Tạo file `.env` trong thư mục gốc theme:

```env
# Development Mode
JANKX_ENV=development
JANKX_DEBUG=true

# Cache Settings
JANKX_CACHE_ENABLED=false

# API Keys (tùy chọn)
GOOGLE_FONTS_API_KEY=your_key_here
```

### Cấu Hình Theme

Chỉnh sửa `config/app.php`:

```php
return [
    'providers' => [
        // Service Providers
        \Jankx\Support\Providers\ContentLayoutServiceProvider::class,
        \Jankx\Admin\Providers\AdminServiceProvider::class,
    ],
    'options' => [
        'framework' => 'jankx',
    ],
];
```

---

## Các Bước Đầu Tiên

### 1. Cài Dependencies

```bash
composer install
npm install
```

### 2. Build Assets

```bash
# Development
npm run dev

# Production
npm run build
```

### 3. Chạy Tests

```bash
./vendor/bin/phpunit
```

### 4. Kích Hoạt Theme

Vào WordPress Admin → Giao Diện → Themes → Kích hoạt Jankx

---

## Các Bước Tiếp Theo

- [Tìm hiểu Hệ Thống Layout](../layout-system/)
- [Khám phá Gutenberg Blocks](../gutenberg-blocks/)
- [Tạo Extensions Tùy Chỉnh](../extensions/)

---

## Xử Lý Lỗi

### Các Vấn Đề Thường Gặp

**Vấn đề**: Lỗi `Class not found`
```bash
# Giải pháp: Regenerate autoload
composer dump-autoload
```

**Vấn đề**: Styles không load
```bash
# Giải pháp: Rebuild assets
npm run build
```

**Vấn đề**: Permission denied trên cache
```bash
# Giải pháp: Fix permissions
chmod -R 755 wp-content/themes/jankx/cache/
```

---

## Hỗ Trợ

- 📖 [Tài Liệu](../)
- 🐛 [Issue Tracker](https://github.com/jankx/theme/issues)
- 💬 [Discussions](https://github.com/jankx/theme/discussions)
