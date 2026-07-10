# Template Bundle System

Hệ thống Template Bundle cho phép đóng gói các **Gutenberg template files (FSE HTML)**, **theme presets** (màu sắc, header layout, typography), và **theme options** thành từng "gói giao diện" có thể chọn lựa khi setup website — tương tự Flatsome Setup Wizard.

## Kiến trúc

```
TemplateBundle (Value Object)
    └─ id, name, description, preset, templates, themeOptions, ...
    
TemplateBundleManager (Registry - Singleton)
    └─ loadBundles() → config/template-bundles.php + filter hooks
    └─ getBundle(id), getBundles(), getActiveBundle()
    
TemplateBundleApplier (Service)
    └─ apply(id) → copy templates, apply options, setup pages
    
TemplateBundleServiceProvider
    └─ register vào container, AJAX handlers
```

## Cấu trúc thư mục bundle

Mỗi bundle là 1 thư mục trong `resources/template-bundles/{bundle-id}/`:

```
resources/template-bundles/
├── corporate/
│   ├── templates/
│   │   ├── index.html
│   │   ├── home.html
│   │   ├── page.html
│   │   ├── single.html
│   │   ├── archive.html
│   │   ├── search.html
│   │   └── 404.html
│   ├── parts/
│   │   ├── header.html
│   │   ├── footer.html
│   │   └── sidebar.html
│   ├── assets/          (optional)
│   └── theme.json       (optional)
├── magazine/
├── creative/
└── landing/
```

## Định nghĩa Bundle trong Config

File: `config/template-bundles.php`

```php
<?php

return [
    'corporate' => [
        // --- Thông tin cơ bản ---
        'name'        => __('Corporate', 'jankx'),
        'description' => __('Professional corporate website.', 'jankx'),
        'preview'     => 'https://example.com/preview.jpg',
        'thumbnail'   => null,
        'tags'        => ['business', 'corporate'],
        'priority'    => 10,

        // --- Preset (màu, header, typography) ---
        'preset' => [
            'colors' => [
                'primary'   => '#1e40af',
                'secondary' => '#0ea5e9',
                'link'      => '#1e40af',
                'link_hover' => '#1e3a8a',
                'header_bg' => '#ffffff',
                'header_text' => '#1e293b',
                'footer_bg' => '#0f172a',
                'footer_text' => '#f8fafc',
                'button_bg' => '#1e40af',
                'button_text' => '#ffffff',
            ],
            'header' => 'classic',   // classic | centered | split | topbar
            'typography' => [
                'body' => [
                    'font-family' => 'Inter, sans-serif',
                    'font-size'   => '16',
                    'font-weight' => '400',
                    'line-height' => '1.6',
                    'color'       => '#334155',
                ],
                'heading' => [
                    'font-family' => 'Plus Jakarta Sans, sans-serif',
                    'font-weight' => '700',
                    'color'       => '#0f172a',
                ],
            ],
        ],

        // --- FSE Template files mapping ---
        // key = tên file đích trong theme, value = tên file nguồn trong bundle
        'templates' => [
            'index.html'  => 'index.html',
            'home.html'   => 'home.html',
            'page.html'   => 'page.html',
            'single.html' => 'single.html',
            'archive.html'=> 'archive.html',
            'search.html' => 'search.html',
            '404.html'    => '404.html',
        ],
        'template_parts' => [
            'header.html' => 'header.html',
            'footer.html' => 'footer.html',
            'sidebar.html'=> 'sidebar.html',
        ],

        // --- Theme options sẽ được apply ---
        'theme_options' => [
            'container_width'       => '1200',
            'sidebar_position'      => 'right',
            'enable_sticky_header'  => true,
            'sticky_header_trigger' => 'top',
            'header_type'           => 'normal',
        ],

        // --- Page setup ---
        'page_setup' => [
            'homepage' => 'home',
            'blog'     => 'blog',
            'menu_location' => [
                'primary' => 'Main Menu',
            ],
        ],

        // --- Plugins/Extensions cần thiết ---
        'required_plugins'    => [],
        'required_extensions' => [],
    ],
];
```

## Hook System

### Filters

#### `jankx/template_bundles`
Modify toàn bộ danh sách bundles. Dùng để thêm, sửa, xoá bundle từ child theme hoặc extension.

```php
// Child theme: thêm bundle mới
add_filter('jankx/template_bundles', function ($bundles) {
    $bundles['real-estate'] = [
        'name'        => 'Real Estate',
        'description' => 'Website bất động sản chuyên nghiệp.',
        'tags'        => ['bds', 'property'],
        'preset' => [
            'colors' => [
                'primary' => '#d97706',
                'secondary' => '#047857',
            ],
            'header' => 'topbar',
        ],
        'templates' => [
            'index.html' => 'index.html',
            'home.html'  => 'home.html',
            // ...
        ],
        'template_parts' => [
            'header.html' => 'header.html',
            // ...
        ],
    ];
    return $bundles;
});
```

```php
// Extension: sửa 1 bundle có sẵn
add_filter('jankx/template_bundles', function ($bundles) {
    if (isset($bundles['corporate'])) {
        $bundles['corporate']['preset']['colors']['primary'] = '#7c3aed';
    }
    return $bundles;
});
```

#### `jankx/template_bundle/{id}`
Filter riêng cho từng bundle. Nhận `TemplateBundle` object.

```php
add_filter('jankx/template_bundle/corporate', function ($bundle) {
    // Sửa màu primary
    // $bundle là TemplateBundle object (immutable-like)
    // Trả về array config mới nếu muốn thay đổi
    return $bundle;
});
```

#### `jankx/template_bundle/source_path`
Thay đổi đường dẫn thư mục chứa template files của bundle.

```php
add_filter('jankx/template_bundle/source_path', function ($basePath, $bundleId) {
    if ($bundleId === 'real-estate') {
        return get_stylesheet_directory() . '/resources/real-estate-bundle';
    }
    return $basePath;
}, 10, 2);
```

### Actions

#### `jankx/template_bundles_loaded`
Gọi sau khi tất cả bundles đã được load.

```php
add_action('jankx/template_bundles_loaded', function ($manager) {
    // $manager là TemplateBundleManager instance
    error_log('Loaded ' . $manager->count() . ' template bundles');
});
```

#### `jankx/template_bundle/before_apply`
Gọi trước khi apply bundle.

```php
add_action('jankx/template_bundle/before_apply', function ($bundleId, $bundle) {
    // Backup dữ liệu hiện tại nếu cần
});
```

#### `jankx/template_bundle/after_apply`
Gọi sau khi apply bundle thành công.

```php
add_action('jankx/template_bundle/after_apply', function ($bundleId, $bundle, $results) {
    // Gửi email thông báo, log, v.v.
});
```

#### `jankx/template_bundle/activated`
Gọi khi bundle được set làm active (lưu option).

```php
add_action('jankx/template_bundle/activated', function ($bundleId, $bundle) {
    update_option('my_plugin_last_bundle', $bundleId);
});
```

#### `jankx/template_bundle/reset`
Gọi khi active bundle bị reset.

## Child Theme: Override Bundle Config

Child theme có thể tạo file `config/template-bundles.php` để:

### 1. Replace toàn bộ 1 bundle
```php
<?php
return [
    'corporate' => [
        '__replace' => true,  // Bắt buộc: xoá bundle cũ, dùng bundle mới
        'name' => 'Corporate v2',
        'preset' => [
            'colors' => [
                'primary' => '#ff0000',
                'secondary' => '#00ff00',
            ],
        ],
        'templates' => [
            'home.html' => 'custom-home.html',
            // ...
        ],
        // ...
    ],
];
```

### 2. Merge vào bundle có sẵn (không có `__replace`)
```php
<?php
return [
    'corporate' => [
        // Merge sâu: chỉ override các key được định nghĩa
        'preset' => [
            'colors' => [
                'primary' => '#ff0000',  // Chỉ đổi màu primary
            ],
        ],
    ],
];
```

### 3. Xoá 1 bundle
```php
<?php
return [
    'landing' => [
        '__remove' => true,  // Xoá bundle landing
    ],
];
```

### 4. Thêm bundle mới
```php
<?php
return [
    'my-custom' => [
        'name' => 'My Custom Bundle',
        // ... đầy đủ config như trên
    ],
];
```

## Extension: Override Bundle

Extension dùng filter `jankx/template_bundles` (giống child theme):

```php
class RealEstateExtension extends AbstractExtension {
    public function register_hooks() {
        add_filter('jankx/template_bundles', [$this, 'addRealEstateBundle'], 20);
    }

    public function addRealEstateBundle($bundles) {
        $bundles['real-estate'] = [
            'name'        => 'Bất Động Sản',
            'description' => 'Giao diện chuyên cho website bất động sản.',
            'tags'        => ['real-estate', 'property'],
            'preset' => [
                'colors' => [
                    'primary'   => '#d97706',
                    'secondary' => '#047857',
                ],
                'header' => 'topbar',
            ],
            'templates' => [
                'home.html'              => 'home.html',
                'single-bat-dong-san.html' => 'single-property.html',
                // ...
            ],
            'template_parts' => [
                'header.html' => 'header.html',
                'footer.html' => 'footer.html',
            ],
            'theme_options' => [
                'container_width' => '1400',
                'sidebar_position' => 'left',
            ],
            'required_extensions' => ['real-estate'],
        ];
        return $bundles;
    }
}
```

## Classes API

### `TemplateBundle`
```php
$bundle = $manager->getBundle('corporate');

$bundle->getId();              // 'corporate'
$bundle->getName();            // 'Corporate'
$bundle->getDescription();     // 'Professional corporate website...'
$bundle->getPreset();          // ['colors' => [...], 'header' => 'classic', 'typography' => [...]]
$bundle->getPresetColor('primary', '#default');  // '#1e40af'
$bundle->getHeaderPreset();    // 'classic'
$bundle->getTemplates();       // ['index.html' => 'index.html', ...]
$bundle->getTemplateParts();   // ['header.html' => 'header.html', ...]
$bundle->getThemeOptions();    // ['container_width' => '1200', ...]
$bundle->getPageSetup();       // ['homepage' => 'home', ...]
$bundle->getTags();            // ['business', 'corporate']
$bundle->getPriority();        // 10
$bundle->toArray();            // Full array representation
```

### `TemplateBundleManager`
```php
$manager = TemplateBundleManager::getInstance();
// Hoặc từ container:
$manager = $app->make('template-bundle.manager');

$manager->loadBundles();
$manager->getBundles();              // [TemplateBundle, ...]
$manager->getBundle('corporate');    // TemplateBundle|null
$manager->hasBundle('corporate');    // bool
$manager->count();                   // int
$manager->getActiveBundleId();       // string
$manager->getActiveBundle();         // TemplateBundle|null
$manager->setActiveBundle('corporate'); // bool
$manager->resetActiveBundle();
$manager->getBundlesByTag('business'); // [TemplateBundle, ...]
```

### `TemplateBundleApplier`
```php
$applier = $app->make('template-bundle.applier');
$result = $applier->apply('corporate');

// $result = [
//     'bundle_id' => 'corporate',
//     'success' => true,
//     'steps' => [
//         'templates'     => ['status' => 'success', 'message' => 'Copied 7 template(s).'],
//         'template_parts'=> ['status' => 'success', 'message' => 'Copied 3 part(s).'],
//         'theme_options' => ['status' => 'success', 'message' => 'Theme options applied.'],
//         'theme_json'    => ['status' => 'success', 'message' => 'theme.json settings applied.'],
//         'page_setup'    => ['status' => 'success', 'message' => 'Page setup applied.'],
//     ],
// ];
```

## AJAX API

### `jankx_get_template_bundles`
Lấy danh sách bundles.

```js
$.post(ajaxurl, {
    action: 'jankx_get_template_bundles',
    nonce: bundleNonce
}, function(response) {
    console.log(response.data.bundles);
    console.log(response.data.active_bundle);
});
```

### `jankx_apply_template_bundle`
Apply 1 bundle.

```js
$.post(ajaxurl, {
    action: 'jankx_apply_template_bundle',
    bundle: 'corporate',
    nonce: bundleNonce
}, function(response) {
    if (response.success) {
        console.log('Applied!', response.data);
    }
});
```

### `jankx_reset_template_bundle`
Reset active bundle.

```js
$.post(ajaxurl, {
    action: 'jankx_reset_template_bundle',
    nonce: bundleNonce
}, function(response) {
    console.log('Reset!', response.data);
});
```

## Setup Wizard

Setup Wizard tự động xuất hiện khi theme được kích hoạt lần đầu (trên Dashboard).

- **Step 1**: Welcome
- **Step 2**: Chọn Template Bundle (hiển thị màu sắc + header preview)
- **Step 3**: Branding (site title, tagline, primary/secondary color) + Apply bundle
- **Step 4**: Done

Có thể skip wizard bất kỳ lúc nào bằng nút "Skip Setup".

Để tuỳ chỉnh wizard: dùng filter `jankx/template_bundles` để thay đổi danh sách bundles hiển thị.

## Tạo Bundle Template Files Mới

1. Tạo thư mục: `resources/template-bundles/{bundle-id}/`
2. Thêm file HTML templates vào `templates/` (FSE block markup)
3. Thêm file HTML template parts vào `parts/`
4. (Optional) Thêm `theme.json` để override theme.json settings
5. (Optional) Thêm `assets/` chứa images, CSS, JS đi kèm bundle
6. Định nghĩa bundle trong `config/template-bundles.php`
