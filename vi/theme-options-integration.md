# 🎨 Hệ Thống Tích Hợp Theme Options

Hệ Thống Tích Hợp Theme Options kết nối Theme Options, Gutenberg Blocks và Global CSS thành một hệ thống đồng bộ thống nhất. Thay đổi trong Theme Options tự động lan tỏa đến tất cả blocks và CSS trên toàn site.

![Pro](https://img.shields.io/badge/PRO-764ba2?style=flat-square)

---

## 🎯 Vấn Đề Đã Giải Quyết

**Trước đây:** Theme Options, Gutenberg Blocks và Global CSS hoạt động độc lập.
- User thay đổi màu trong Theme Options → Blocks không nhận được màu mới
- Container width trong Theme Options khác với layout trong theme.json
- Typography settings không áp dụng cho blocks

**Sau khi tích hợp:** Hệ thống hoạt động đồng bộ
- Thay đổi ở Theme Options → Tự động cập nhật CSS và Blocks
- CSS variables kết nối mọi thành phần

---

## 🏗️ Kiến Trúc

```
┌─────────────────────────────────────────────────────────────┐
│                     THEME OPTIONS                           │
│  (primary_color, container_width, body_typography...)       │
└──────────────────────┬──────────────────────────────────────┘
                       │
                       ▼
┌─────────────────────────────────────────────────────────────┐
│          ThemeOptionsCSSGenerator                           │
│  Tạo CSS Variables:                                         │
│  --jankx-primary-color, --jankx-container-width...          │
└──────────────────────┬──────────────────────────────────────┘
                       │
         ┌─────────────┴─────────────┐
         │                           │
         ▼                           ▼
┌────────────────────┐    ┌─────────────────────────────┐
│  GLOBAL CSS        │    │  ThemeOptionsBridge         │
│  (wp_add_inline_   │    │                             │
│   style)           │    │  • Truyền data vào JS       │
│                    │    │  • Filter theme.json        │
│  Frontend + Admin  │    │  • Body classes             │
│  + Block Editor    │    │  • Block defaults           │
└────────┬───────────┘    └────────────┬────────────────┘
         │                            │
         └────────────┬───────────────┘
                      │
                      ▼
         ┌────────────────────────┐
         │    GUTENBERG BLOCKS    │
         │                        │
         │  • CSS Variables       │
         │  • window.jankxTheme   │
         │    Options             │
         │  • Preset colors       │
         └────────────────────────┘
```

---

## 📂 Các Thành Phần Hệ Thống

| File | Chức Năng | Khi Nào Chạy |
|------|-----------|--------------|
| `includes/app/Services/ThemeOptionsCSSGenerator.php` | Tạo CSS từ theme options | Mỗi khi load trang |
| `includes/app/Services/ThemeOptionsBridge.php` | Kết nối với blocks và theme.json | Mỗi khi load trang |
| `includes/app/Providers/ThemeOptionsIntegrationServiceProvider.php` | Đăng ký services | Khởi động theme |
| `includes/app/helpers/theme-options.php` | Helper functions | Theo yêu cầu |

---

## 🎨 CSS Variables Được Tạo

Khi user thay đổi theme options, các CSS variables này sẽ được generate:

```css
:root {
  /* Colors */
  --jankx-primary-color: #ff5722;      /* từ primary_color */
  --jankx-secondary-color: #009688;   /* từ secondary_color */
  --jankx-primary-color-rgb: 255, 87, 34;
  --jankx-secondary-color-rgb: 0, 150, 136;

  /* Layout */
  --jankx-container-width: 1200px;    /* từ container_width */
  --jankx-sidebar-position: right;    /* từ sidebar_position */

  /* Typography */
  --jankx-body-font-family: Inter;    /* từ body_typography */
  --jankx-body-font-size: 16px;
  --jankx-body-font-weight: 400;
  --jankx-body-line-height: 1.6;
  --jankx-body-text-color: #222222;
}
```

---

## 💻 Cách Sử Dụng Cho Developer

### Trong PHP Templates

```php
<?php
// Lấy giá trị theme option
$primaryColor = jankx_get_theme_option('primary_color', '#ff5722');

// Lấy màu với các định dạng khác nhau
$hex = jankx_get_theme_color('primary', 'hex');        // #ff5722
$rgb = jankx_get_theme_color('primary', 'rgb');        // 255, 87, 34
$css = jankx_get_theme_color('primary', 'css-var');   // var(--jankx-primary-color)

// Lấy container width
$width = jankx_get_container_width();                  // 1200px
$widthValue = jankx_get_container_width(false);       // 1200 (số)

// Lấy typography
$font = jankx_get_body_typography('font-family');     // Inter
$allTypography = jankx_get_body_typography();         // Array tất cả

// Lấy CSS variable
$var = jankx_get_css_var('primary-color');            // var(--jankx-primary-color)
$value = jankx_get_css_var('primary-color', true);     // #ff5722 (giá trị thực)
?>

<!-- Sử dụng trong HTML -->
<div style="background-color: <?php echo esc_attr($css); ?>; max-width: <?php echo $width; ?>;">
    Content
</div>
```

### Trong Block Rendering (PHP)

```php
<?php
namespace Jankx\Gutenberg\Blocks;

class MyBlock extends Block {

    public function render($attributes, $content, $block) {
        // Lấy theme colors
        $primaryColor = jankx_get_theme_color('primary', 'css-var');
        $containerWidth = jankx_get_css_var('container-width');

        // Tạo inline styles
        $styles = sprintf(
            'background-color: %s; max-width: %s;',
            $primaryColor,
            $containerWidth
        );

        // Hoặc dùng trực tiếp CSS variables
        $styles = 'background-color: var(--jankx-primary-color);';

        return sprintf(
            '<div class="my-block" style="%s">%s</div>',
            esc_attr($styles),
            $content
        );
    }
}
```

### Trong JavaScript (Block Editor)

```javascript
// Truy cập theme options trong block
const themeOptions = window.jankxThemeOptions;

// Cấu trúc data:
// {
//   version: '1.0.0',
//   colors: {
//     primary: '#ff5722',
//     secondary: '#009688'
//   },
//   layout: {
//     containerWidth: 1200,
//     sidebarPosition: 'right'
//   },
//   typography: {
//     body: {
//       'font-family': 'Inter',
//       'font-size': '16px',
//       ...
//     }
//   }
// }

// Ví dụ: Block edit component
export default function Edit({ attributes, setAttributes }) {
    const themeOptions = window.jankxThemeOptions || {};
    const primaryColor = themeOptions.colors?.primary || '#ff5722';

    return (
        <div style={{
            backgroundColor: primaryColor,
            maxWidth: themeOptions.layout?.containerWidth + 'px' || '1200px'
        }}>
            <p style={{
                fontFamily: themeOptions.typography?.body?.['font-family'] || 'Inter'
            }}>
                Block content
            </p>
        </div>
    );
}
```

### Trong SCSS/CSS

```scss
.my-block {
  // Sử dụng CSS variables
  background-color: var(--jankx-primary-color, #ff5722);
  color: var(--jankx-secondary-color, #009688);
  max-width: var(--jankx-container-width, 1200px);
  font-family: var(--jankx-body-font-family, Inter);
  font-size: var(--jankx-body-font-size, 16px);

  // Responsive
  @media (max-width: 768px) {
    max-width: 100%;
  }
}

// Sử dụng RGB variables cho opacity
.overlay {
  background-color: rgba(
    var(--jankx-primary-color-rgb),
    0.5  // 50% opacity
  );
}
```

### Trong theme.json

Đã cập nhật để dùng CSS variables làm fallback:

```json
{
  "styles": {
    "typography": {
      "fontFamily": "var(--jankx-body-font-family, var(--wp--preset--font-family--inter))",
      "fontSize": "var(--jankx-body-font-size, 15px)",
      "lineHeight": "var(--jankx-body-line-height, 1.6)"
    },
    "elements": {
      "link": {
        ":hover": {
          "color": {
            "text": "var(--jankx-primary-color, var(--wp--preset--color--primary))"
          }
        }
      }
    }
  }
}
```

---

## 👤 Cách Sử Dụng Cho End User (Không Kỹ Thuật)

### Thay Đổi Màu Sắc

1. Vào **Giao Diện → Theme Options → Branding**
2. Thay đổi **Primary Color** và **Secondary Color**
3. Click **Lưu Thay Đổi**
4. **Tự động:**
   - Các blocks dùng màu primary sẽ cập nhật
   - Links hover sẽ đổi màu
   - Theme palette sẽ đồng bộ

### Thay Đổi Kích Thước Container

1. Vào **Giao Diện → Theme Options → Layout**
2. Điều chỉnh **Container Width** (960px - 1440px)
3. Click **Lưu Thay Đổi**
4. **Tự động:**
   - Layout toàn site thay đổi
   - Gutenberg editor cũng cập nhật
   - Blocks giữ nguyên tỷ lệ

### Thay Đổi Font Chữ

1. Vào **Giao Diện → Theme Options → Typography**
2. Chọn **Body Font**
3. Thay đổi các thông số: Font family, Size, Weight, Line height
4. Click **Lưu Thay Đổi**
5. **Tự động:**
   - Font toàn site thay đổi
   - Blocks dùng body font sẽ cập nhật
   - Editor cũng hiển thị font mới

### Thay Đổi Vị Trí Sidebar

1. Vào **Giao Diện → Theme Options → Layout**
2. Chọn **Sidebar Position**: Right / Left / No Sidebar
3. Click **Lưu Thay Đổi**
4. **Tự động:**
   - Layout chuyển sang trái/phải/không sidebar
   - Body class thay đổi (`jankx-sidebar-right`, `jankx-sidebar-left`, `jankx-sidebar-none`)
   - CSS responsive tự điều chỉnh

---

## 🔧 Tích Hợp Cho Block Mới

Khi tạo block mới, làm theo các bước sau:

### Bước 1: Trong PHP Block Class

```php
<?php
class NewBlock extends Block {
    protected $blockId = 'jankx/new-block';

    public function enqueueEditorAssets(): void {
        parent::enqueueEditorAssets();

        // Truyền theme options vào JS
        $handle = 'jankx-new-block-editor-script';

        if (function_exists('jankx_get_theme_options_data')) {
            wp_localize_script(
                $handle,
                'jankxNewBlockThemeOptions',
                jankx_get_theme_options_data()
            );
        }
    }

    public function render($attributes, $content, $block) {
        // Lấy theme colors
        $primary = jankx_get_theme_color('primary', 'css-var');
        $secondary = jankx_get_theme_color('secondary', 'css-var');

        // Tạo inline styles
        $styles = sprintf(
            '--block-primary: %s; --block-secondary: %s;',
            $primary,
            $secondary
        );

        return sprintf(
            '<div class="wp-block-jankx-new-block" style="%s">%s</div>',
            esc_attr($styles),
            $content
        );
    }
}
```

### Bước 2: Trong block.json

```json
{
  "name": "jankx/new-block",
  "attributes": {
    "useThemeColors": {
      "type": "boolean",
      "default": true
    }
  },
  "supports": {
    "color": {
      "text": true,
      "background": true,
      "link": true
    }
  }
}
```

### Bước 3: Trong edit.js (React)

```javascript
import { useSelect } from '@wordpress/data';

export default function Edit({ attributes, setAttributes }) {
    // Lấy theme options từ window object
    const themeOptions = window.jankxNewBlockThemeOptions || {};
    const { useThemeColors } = attributes;

    // Hoặc lấy từ block editor settings
    const colors = useSelect((select) => {
        return select('core/block-editor').getSettings().colors;
    }, []);

    // Tìm primary color từ palette
    const primaryColor = colors?.find(c => c.slug === 'primary')?.color
        || themeOptions.colors?.primary
        || '#ff5722';

    return (
        <div
            className="jankx-new-block"
            style={{
                backgroundColor: useThemeColors
                    ? primaryColor
                    : attributes.backgroundColor
            }}
        >
            {/* Block content */}
        </div>
    );
}
```

### Bước 4: Trong style.scss

```scss
.wp-block-jankx-new-block {
  // Sử dụng CSS variables từ theme
  background-color: var(--block-primary, var(--jankx-primary-color, #ff5722));
  color: var(--block-secondary, var(--jankx-secondary-color, #009688));
  max-width: var(--jankx-container-width, 1200px);
  font-family: var(--jankx-body-font-family, Inter);

  // Preset styles
  &.is-style-default {
    padding: 2rem;
  }

  &.is-style-compact {
    padding: 1rem;
  }
}
```

---

## 🧪 Kiểm Tra Hệ Thống

### Cách 1: Dùng WP-CLI

```bash
# Chạy test script
wp eval-file wp-content/themes/jankx/tests/theme-options-integration-test.php
```

### Cách 2: Kiểm Tra Thủ Công

**Trong Admin:**
1. Mở **Giao Diện → Theme Options**
2. Thay đổi **Primary Color** thành màu khác (ví dụ: #ff0000)
3. Lưu thay đổi
4. Mở **Trang → Thêm Mới** (Block Editor)
5. Kiểm tra:
   - Màu primary trong color palette có đổi thành #ff0000 không?
   - Blocks dùng màu primary có cập nhật không?

**Trên Frontend:**
1. Mở DevTools (F12)
2. Vào tab **Elements**
3. Tìm `<style id="jankx-theme-options-css">`
4. Kiểm tra CSS variables:
   ```css
   :root {
     --jankx-primary-color: #ff0000;  // Màu bạn vừa đổi
   }
   ```

**Trong Console:**
```javascript
// Kiểm tra theme options data
console.log(window.jankxThemeOptions);

// Kết quả mong đợi:
// {
//   colors: { primary: '#ff0000', secondary: '...' },
//   layout: { containerWidth: 1200, ... },
//   typography: { body: { ... } }
// }
```

---

## 🐛 Xử Lý Lỗi

### Vấn Đề 1: CSS Không Cập Nhật Khi Đổi Theme Options

**Nguyên nhân:** Cache

**Giải pháp:**
```php
// Trong functions.php hoặc custom plugin
add_action('jankx/options/updated', function() {
    // Clear CSS cache
    do_action('jankx/theme-options-integration/clear-cache');

    // Hoặc dùng plugin cache clear
    if (function_exists('wp_cache_flush')) {
        wp_cache_flush();
    }
});
```

### Vấn Đề 2: Blocks Không Nhận Theme Colors

**Nguyên nhân:** Block chưa được tích hợp

**Giải pháp:**
- Thêm `window.jankxThemeOptions` vào block's editor script
- Dùng helper functions trong block's render method

### Vấn Đề 3: theme.json Không Đồng Bộ

**Kiểm tra:**
```php
// Trong functions.php
do_action('wp_theme_json_data_theme', function($theme_json) {
    error_log(print_r($theme_json->get_data(), true));
    return $theme_json;
});
```

---

## 📋 Checklist Tích Hợp Cho Block Mới

- [ ] Thêm `jankx_get_theme_options_data()` vào `enqueueEditorAssets()`
- [ ] Dùng `jankx_get_theme_color()` trong `render()` method
- [ ] Thêm CSS variables trong `style.scss`
- [ ] Test với thay đổi theme options
- [ ] Kiểm tra responsive
- [ ] Kiểm tra trong editor và frontend

---

## 📚 Tài Liệu Liên Quan

- **Theme Options Location:** `Giao Diện → Theme Options`
- **English Documentation:** `docs/THEME_OPTIONS_INTEGRATION.md`
- **Test Script:** `tests/theme-options-integration-test.php`
- **Helper Functions:** `includes/app/helpers/theme-options.php`

---

<p align="center">
  <a href="./">← Quay lại Tài Liệu</a>
</p>
