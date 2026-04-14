# Hệ Thống Layout

Hệ Thống Layout của Jankx cung cấp cách quản lý layout động mạnh mẽ và linh hoạt.

## Tổng Quan

```
┌─────────────────────────────────────────┐
│         Layout Registry                 │
│  ┌─────────┐ ┌─────────┐ ┌─────────┐  │
│  │  Grid   │ │  List   │ │Carousel │  │
│  │ Layout  │ │ Layout  │ │ Layout  │  │
│  └────┬────┘ └────┬────┘ └────┬────┘  │
│       └────────────┴────────────┘       │
│              BlockTemplate              │
└─────────────────────────────────────────┘
```

## Mục Lục

- [Khái Niệm Cốt Lõi](#khái-niệm-cốt-lõi)
- [Layout Registry](#layout-registry)
- [Các Layout Có Sẵn](#các-layout-có-sẵn)
- [Tạo Layout Tùy Chỉnh](#tạo-layout-tùy-chỉnh)

---

## Khái Niệm Cốt Lõi

### Layout Registry

```php
use Jankx\Layouts\DynamicDataLayout\LayoutRegistry;
use Jankx\Foundation\Application;

$app = Application::getInstance();
$registry = new LayoutRegistry($app);

// Đăng ký layout
$registry->register('grid', GridLayout::class);

// Kiểm tra và resolve
if ($registry->has('grid')) {
    $layout = $registry->resolve('grid', ['columns' => 3]);
}
```

---

## Layout Registry

### Các Phương Thức

| Phương Thức | Mô Tả |
|-------------|-------|
| `register(string $name, string $class)` | Đăng ký layout mới |
| `unregister(string $name)` | Xóa layout |
| `has(string $name)` | Kiểm tra layout tồn tại |
| `get(string $name)` | Lấy tên class layout |
| `resolve(string $name, array $options)` | Tạo instance layout |
| `all()` | Lấy tất cả layouts |
| `getNames()` | Lấy tên tất cả layouts |

---

## Các Layout Có Sẵn

### Grid Layout

```php
use Jankx\Layouts\DynamicDataLayout\BlockLayouts\GridLayout;

$layout = new GridLayout();
$layout->setOptions([
    'columns' => 3,
    'gap' => 20,
    'class_name' => 'my-grid',
]);

echo $layout->render($data);
```

**Tùy chọn:**
- `columns` (int): Số cột (1-12)
- `gap` (int): Khoảng cách giữa items (px)
- `responsive` (bool): Bật responsive
- `class_name` (string): Tên class CSS

### List Layout

```php
use Jankx\Layouts\DynamicDataLayout\BlockLayouts\ListLayout;

$layout = new ListLayout();
$layout->setOptions([
    'style' => 'vertical',
    'class_name' => 'my-list',
]);
```

### Carousel Layout ![Pro](https://img.shields.io/badge/PRO-764ba2?style=flat-square)

```php
use Jankx\Layouts\DynamicDataLayout\BlockLayouts\CarouselLayout;

$layout = new CarouselLayout();
$layout->setOptions([
    'slides_per_view' => 3,
    'autoplay' => true,
    'navigation' => true,
]);
```

---

## Tạo Layout Tùy Chỉnh

### Bước 1: Tạo Class Layout

```php
<?php
namespace MyTheme\Layouts;

use Jankx\Layouts\DynamicDataLayout\BlockTemplateLayout;

class MasonryLayout extends BlockTemplateLayout
{
    protected $defaultOptions = [
        'columns' => 3,
        'gap' => 15,
        'class_name' => 'masonry-layout',
    ];

    public function getHtmlStructure(): string
    {
        return '<div class="{{class_name}} masonry-grid">
            {{#items}}
                <div class="masonry-item" style="margin-bottom: {{gap}}px;">
                    {{content}}
                </div>
            {{/items}}
        </div>';
    }
}
```

### Bước 2: Đăng Ký Layout

```php
add_action('jankx/layouts/register', function(LayoutRegistry $registry) {
    $registry->register('masonry', MasonryLayout::class);
});
```

---

## Best Practices

1. **Luôn dùng LayoutRegistry**: Không khởi tạo layout trực tiếp
2. **Validate options**: Dùng `setOptions()` với type checking
3. **Cache renders**: Cache output cho layout phức tạp
4. **Responsive design**: Dùng options responsive có sẵn
5. **Accessibility**: Bao gồm ARIA labels và semantic HTML
