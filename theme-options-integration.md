# Theme Options Integration Guide

Hệ thống tích hợp Theme Options, Gutenberg Blocks và Global CSS đã được triển khai.

## Cấu trúc hệ thống

```
Theme Options (User Settings)
    ↓
ThemeOptionsCSSGenerator → CSS Variables (--jankx-*)
    ↓
ThemeOptionsBridge → Gutenberg Blocks + theme.json
    ↓
Global CSS ←→ Block Styles ←→ Editor Styles
```

## Các thành phần chính

### 1. ThemeOptionsCSSGenerator
- File: `includes/app/Services/ThemeOptionsCSSGenerator.php`
- Chức năng: Tạo CSS động từ theme options và inject vào site
- CSS Variables được tạo:
  - `--jankx-primary-color`
  - `--jankx-secondary-color`
  - `--jankx-container-width`
  - `--jankx-body-font-family`
  - `--jankx-body-font-size`
  - `--jankx-body-font-weight`
  - `--jankx-body-line-height`
  - `--jankx-body-text-color`
  - `--jankx-sidebar-position`

### 2. ThemeOptionsBridge
- File: `includes/app/Services/ThemeOptionsBridge.php`
- Chức năng:
  - Truyền theme options vào Gutenberg Blocks qua JavaScript (`window.jankxThemeOptions`)
  - Đồng bộ theme options với theme.json (colors, layout)
  - Thêm body classes dựa trên theme options
  - Áp dụng theme defaults vào block rendering

### 3. ThemeOptionsIntegrationServiceProvider
- File: `includes/app/Providers/ThemeOptionsIntegrationServiceProvider.php`
- Đăng ký và khởi tạo các services

### 4. Helper Functions
- File: `includes/app/helpers/theme-options.php`
- Các hàm tiện ích:
  - `jankx_get_theme_option($key, $default)`
  - `jankx_get_theme_color($type, $format)`
  - `jankx_get_container_width($withUnit)`
  - `jankx_get_body_typography($property)`
  - `jankx_get_css_var($varName, $returnValue)`
  - `jankx_get_theme_options_data()`
  - `jankx_render_inline_theme_css()`

## Cách sử dụng

### Trong PHP Templates

```php
// Lấy giá trị theme option
$primaryColor = jankx_get_theme_option('primary_color', '#ff5722');

// Lấy màu với định dạng khác nhau
$primaryHex = jankx_get_theme_color('primary', 'hex');        // #ff5722
$primaryRgb = jankx_get_theme_color('primary', 'rgb');        // 255, 87, 34
$primaryVar = jankx_get_theme_color('primary', 'css-var');    // var(--jankx-primary-color)

// Lấy CSS variable
$containerWidth = jankx_get_css_var('container-width');       // var(--jankx-container-width)
$actualWidth = jankx_get_css_var('container-width', true);    // 1200px

// Lấy typography
$bodyFont = jankx_get_body_typography('font-family');          // Inter
$allTypography = jankx_get_body_typography();                  // Array of all values
```

### Trong Block Rendering (PHP)

```php
class MyCustomBlock extends Block {
    public function render($attributes, $content, $block) {
        $primaryColor = jankx_get_theme_color('primary', 'css-var');
        $containerWidth = jankx_get_css_var('container-width');

        $styles = sprintf(
            'background-color: %s; max-width: %s;',
            $primaryColor,
            $containerWidth
        );

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
// Truy cập theme options trong block editor
const themeOptions = window.jankxThemeOptions;

// Ví dụ: Lấy màu primary
const primaryColor = themeOptions.colors.primary;

// Ví dụ: Typography
const bodyFont = themeOptions.typography.body['font-family'];

// Ví dụ: Layout
const containerWidth = themeOptions.layout.containerWidth;

// Sử dụng trong block edit component
export default function Edit({ attributes, setAttributes }) {
    const themeOptions = window.jankxThemeOptions || {};

    return (
        <div style={{
            backgroundColor: themeOptions.colors?.primary || '#ff5722',
            maxWidth: themeOptions.layout?.containerWidth + 'px' || '1200px'
        }}>
            Block content
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

    // Responsive với CSS variables
    @media (max-width: 768px) {
        max-width: 100%;
    }
}
```

### Trong theme.json

Theme.json đã được cập nhật để sử dụng CSS variables làm fallback:

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

## Quy trình hoạt động

1. **Khi theme được load:**
   - ThemeOptionsService khởi tạo từ config
   - ThemeOptionsCSSGenerator tạo CSS và enqueue
   - ThemeOptionsBridge truyền data vào JS

2. **Khi user thay đổi theme options:**
   - Options được lưu vào database
   - Cache được clear
   - CSS được regenerate ở lần load tiếp theo

3. **Khi block editor mở:**
   - CSS variables được inject vào editor iframe
   - `window.jankxThemeOptions` chứa data để blocks sử dụng
   - theme.json palette được filter để match theme options

4. **Khi frontend render:**
   - CSS inline được enqueue với các CSS variables
   - Blocks sử dụng CSS variables cho styling
   - Helper functions có thể truy cập trực tiếp theme options

## Hook và Filters

```php
// Filter để enable/disable frontend enqueue
add_filter('jankx/theme_options/enqueue_frontend', function($enabled) {
    return $enabled;
});

// Action khi integration được khởi tạo
add_action('jankx/theme-options-integration/initialized', function() {
    // Do something after integration is ready
});

// Filter theme.json data
add_filter('wp_theme_json_data_theme', function($themeJson) {
    // Modify theme.json data
    return $themeJson;
});
```

## Debug

```php
// Kiểm tra theme options data
$themeData = jankx_get_theme_options_data();
var_dump($themeData);

// Kiểm tra CSS variables
$cssVars = jankx_get_css_var('primary-color', true);
echo $cssVars;

// Render inline CSS để kiểm tra
jankx_render_inline_theme_css();
```

## Lưu ý

- CSS variables được ưu tiên hơn theme.json defaults
- Khi theme option không set, sẽ sử dụng giá trị mặc định từ config
- Changes trong theme options sẽ tự động cập nhật CSS mà không cần rebuild
- Body class `jankx-sidebar-{position}` được thêm để hỗ trợ layout styling
