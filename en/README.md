# Jankx Theme Documentation

<p align="center">
  <img src="https://raw.githubusercontent.com/jankx/theme/main/assets/logo.png" alt="Jankx Theme Logo" width="120">
</p>

<p align="center">
  <strong>A Modern, Flexible WordPress Theme Framework</strong>
</p>

<p align="center">
  <span style="background: linear-gradient(135deg, #667eea 0%, #764ba2 100%); color: white; padding: 4px 12px; border-radius: 4px; font-weight: bold;">PRO VERSION</span>
</p>

<p align="center">
  <a href="#quick-start">Quick Start</a> •
  <a href="#architecture">Architecture</a> •
  <a href="#api-reference">API Reference</a> •
  <a href="#contributing">Contributing</a>
</p>

---

## 📚 Documentation Structure

| Section | Description | Availability |
|---------|-------------|:------------:|
| **[Getting Started](./getting-started/)** | Installation, setup, and quick start guide | ![Lite](https://img.shields.io/badge/LITE-667eea?style=flat-square) ![Pro](https://img.shields.io/badge/PRO-764ba2?style=flat-square) |
| **[Layout System](./layout-system/)** | Dynamic layouts, block templates, and layout registry | ![Lite](https://img.shields.io/badge/LITE-667eea?style=flat-square) ![Pro](https://img.shields.io/badge/PRO-764ba2?style=flat-square) |
| **[Admin](./admin/)** | Admin panels, form handlers, and theme options | ![Lite](https://img.shields.io/badge/LITE-667eea?style=flat-square) ![Pro](https://img.shields.io/badge/PRO-764ba2?style=flat-square) |
| **[Extensions](./extensions/)** | Extension system, manifest, and lifecycle | ![Lite](https://img.shields.io/badge/LITE-667eea?style=flat-square) ![Pro](https://img.shields.io/badge/PRO-764ba2?style=flat-square) |
| **[Services](./services/)** | Fonts, icons, and theme services | ![Pro](https://img.shields.io/badge/PRO-764ba2?style=flat-square) |
| **[Testing](./testing/)** | Unit tests, integration tests, and coverage | ![Lite](https://img.shields.io/badge/LITE-667eea?style=flat-square) ![Pro](https://img.shields.io/badge/PRO-764ba2?style=flat-square) |

---

## 🚀 Quick Start

```bash
# Clone the repository
git clone https://github.com/jankx/theme.git

# Install dependencies
composer install
npm install

# Run tests
./vendor/bin/phpunit

# Build assets
npm run build
```

---

## 🏗️ Architecture

```
Jankx Theme
├── includes/
│   ├── framework/          # Core framework
│   │   ├── Admin/          # Admin handlers
│   │   ├── Extensions/     # Extension system
│   │   ├── Layouts/        # Layout management
│   │   ├── Services/       # Theme services (Pro)
│   │   └── Support/        # Support classes
│   └── app/                # Application layer
│       ├── Providers/      # Service providers
│       └── Services/       # App services
├── assets/                 # Static assets
├── tests/                  # Test suites
└── docs/                   # Documentation
```

---

## 🏷️ Feature Availability

Features marked with badges:

- ![Lite](https://img.shields.io/badge/LITE-667eea?style=flat-square) - Available in Lite version
- ![Pro](https://img.shields.io/badge/PRO-764ba2?style=flat-square) - Pro version only
- Both badges - Available in all versions

---

## 🧪 Testing

- **Unit Tests**: `tests/` - PHPUnit test suites
- **Integration Tests**: `tests/Integration/` - Full integration tests
- **Code Coverage**: Run with Xdebug for coverage reports

```bash
# Run all tests
./vendor/bin/phpunit

# Run specific test suite
./vendor/bin/phpunit --filter="FormHandlerTest"

# Generate coverage report
./vendor/bin/phpunit --coverage-html coverage/
```

---

## 🤝 Contributing

Please read our [Contributing Guide](./contributing.md) for details on our code of conduct and development process.

---

<p align="center">
  <a href="../">← Back to Language Selection</a>
</p>

<p align="center">
  Made with ❤️ by the Jankx Team
</p>
