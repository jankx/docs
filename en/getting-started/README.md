# Getting Started with Jankx Theme

Welcome to Jankx Theme! This guide will help you get up and running quickly.

## Table of Contents

- [Installation](#installation)
- [Requirements](#requirements)
- [Configuration](#configuration)
- [First Steps](#first-steps)

---

## Installation

### Method 1: Via Composer (Recommended)

```bash
composer require jankx/theme
```

### Method 2: Manual Installation

1. Download the latest release from [GitHub Releases](https://github.com/jankx/jankx-lite/releases)
2. Extract to `/wp-content/themes/jankx/`
3. Activate in WordPress Admin → Appearance → Themes

### Method 3: Git Clone

```bash
cd /wp-content/themes/
git clone https://github.com/jankx/jankx-lite.git jankx
```

---

## Requirements

| Requirement | Version |
|-------------|---------|
| PHP | >= 7.4 |
| WordPress | >= 5.8 |
| Composer | >= 2.0 |
| Node.js | >= 16.0 (for development) |

---

## Configuration

### Environment Setup

Create `.env` file in theme root:

```env
# Development Mode
JANKX_ENV=development
JANKX_DEBUG=true

# Cache Settings
JANKX_CACHE_ENABLED=false

# API Keys (optional)
GOOGLE_FONTS_API_KEY=your_key_here
```

### Theme Configuration

Edit `config/app.php`:

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

## First Steps

### 1. Install Dependencies

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

### 3. Run Tests

```bash
./vendor/bin/phpunit
```

### 4. Activate Theme

Go to WordPress Admin → Appearance → Themes → Activate Jankx

---

## Next Steps

- [Learn about the Layout System](../layout-system/)
- [Explore Gutenberg Blocks](../gutenberg-blocks/)
- [Create Custom Extensions](../extensions/)

---

## Troubleshooting

### Common Issues

**Issue**: `Class not found` errors
```bash
# Solution: Regenerate autoload
composer dump-autoload
```

**Issue**: Styles not loading
```bash
# Solution: Rebuild assets
npm run build
```

**Issue**: Permission denied on cache
```bash
# Solution: Fix permissions
chmod -R 755 wp-content/themes/jankx/cache/
```

---

## Support

- 📖 [Documentation](../)
- 🐛 [Issue Tracker](https://github.com/jankx/jankx-lite/issues)
- 💬 [Discussions](https://github.com/jankx/jankx-lite/discussions)
