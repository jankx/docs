# Tài Liệu Jankx Theme

<p align="center">
  <img src="https://raw.githubusercontent.com/jankx/theme/main/assets/logo.png" alt="Jankx Theme Logo" width="120">
</p>

<p align="center">
  <strong>Theme WordPress Hiện Đại, Linh Hoạt</strong>
</p>

<p align="center">
  <span style="background: linear-gradient(135deg, #667eea 0%, #764ba2 100%); color: white; padding: 4px 12px; border-radius: 4px; font-weight: bold;">PHIÊN BẢN PRO</span>
</p>

<p align="center">
  <a href="#bắt-đầu-nhanh">Bắt Đầu Nhanh</a> •
  <a href="#kiến-trúc">Kiến Trúc</a> •
  <a href="#api-reference">API Reference</a> •
  <a href="#đóng-góp">Đóng Góp</a>
</p>

---

## 📚 Cấu Trúc Tài Liệu

| Phần | Mô Tả | Khả Dụng |
|------|-------|:--------:|
| **[Bắt Đầu](./getting-started/)** | Cài đặt, thiết lập và hướng dẫn nhanh | ![Lite](https://img.shields.io/badge/LITE-667eea?style=flat-square) ![Pro](https://img.shields.io/badge/PRO-764ba2?style=flat-square) |
| **[Hệ Thống Layout](./layout-system/)** | Layout động, block templates, và layout registry | ![Lite](https://img.shields.io/badge/LITE-667eea?style=flat-square) ![Pro](https://img.shields.io/badge/PRO-764ba2?style=flat-square) |
| **[Quản Trị](./admin/)** | Admin panels, form handlers, và theme options | ![Lite](https://img.shields.io/badge/LITE-667eea?style=flat-square) ![Pro](https://img.shields.io/badge/PRO-764ba2?style=flat-square) |
| **[Extensions](./extensions/)** | Hệ thống extension, manifest, và vòng đời | ![Lite](https://img.shields.io/badge/LITE-667eea?style=flat-square) ![Pro](https://img.shields.io/badge/PRO-764ba2?style=flat-square) |
| **[Services](./services/)** | Fonts, icons, và theme services | ![Pro](https://img.shields.io/badge/PRO-764ba2?style=flat-square) |
| **[Testing](./testing/)** | Unit tests, integration tests, và coverage | ![Lite](https://img.shields.io/badge/LITE-667eea?style=flat-square) ![Pro](https://img.shields.io/badge/PRO-764ba2?style=flat-square) |

---

## 🚀 Bắt Đầu Nhanh

```bash
# Clone repository
git clone https://github.com/jankx/theme.git

# Cài dependencies
composer install
npm install

# Chạy tests
./vendor/bin/phpunit

# Build assets
npm run build
```

---

## 🏗️ Kiến Trúc

```
Jankx Theme
├── includes/
│   ├── framework/          # Core framework
│   │   ├── Admin/          # Admin handlers
│   │   ├── Extensions/     # Hệ thống extension
│   │   ├── Layouts/        # Quản lý layout
│   │   ├── Services/       # Theme services (Pro)
│   │   └── Support/        # Support classes
│   └── app/                # Application layer
│       ├── Providers/      # Service providers
│       └── Services/       # App services
├── assets/                 # Static assets
├── tests/                  # Test suites
└── docs/                   # Tài liệu
```

---

## 🏷️ Tính Năng Khả Dụng

Các tính năng được đánh dấu bằng badge:

- ![Lite](https://img.shields.io/badge/LITE-667eea?style=flat-square) - Có trong bản Lite
- ![Pro](https://img.shields.io/badge/PRO-764ba2?style=flat-square) - Chỉ có trong bản Pro
- Cả 2 badges - Có trong tất cả phiên bản

---

## 🧪 Kiểm Thử

- **Unit Tests**: `tests/` - PHPUnit test suites
- **Integration Tests**: `tests/Integration/` - Integration tests đầy đủ
- **Code Coverage**: Chạy với Xdebug để báo cáo coverage

```bash
# Chạy tất cả tests
./vendor/bin/phpunit

# Chạy test suite cụ thể
./vendor/bin/phpunit --filter="FormHandlerTest"

# Tạo báo cáo coverage
./vendor/bin/phpunit --coverage-html coverage/
```

---

## 🤝 Đóng Góp

Vui lòng đọc [Hướng Dẫn Đóng Góp](./contributing.md) để biết chi tiết về code of conduct và quy trình phát triển.

---

<p align="center">
  <a href="../">← Quay lại chọn ngôn ngữ</a>
</p>

<p align="center">
  Made with ❤️ by Jankx Team
</p>
