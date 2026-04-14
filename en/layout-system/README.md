# Layout System

The Jankx Layout System provides a powerful, flexible way to manage dynamic layouts for your WordPress theme.

## Overview

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

## Table of Contents

- [Core Concepts](#core-concepts)
- [Layout Registry](#layout-registry)
- [Block Template Layout](#block-template-layout)
- [Available Layouts](#available-layouts)
- [Creating Custom Layouts](#creating-custom-layouts)
- [Usage Examples](#usage-examples)

---

## Core Concepts

### Layout Registry

The `LayoutRegistry` manages all registered layouts and provides dependency injection support:

```php
use Jankx\Layouts\DynamicDataLayout\LayoutRegistry;
use Jankx\Foundation\Application;

$app = Application::getInstance();
$registry = new LayoutRegistry($app);

// Register a layout
$registry->register('grid', GridLayout::class);

// Check if layout exists
if ($registry->has('grid')) {
    $layout = $registry->resolve('grid', ['columns' => 3]);
}
```

### Block Template Layout

Base class for all layouts, providing common functionality:

```php
use Jankx\Layouts\DynamicDataLayout\BlockTemplateLayout;

class CustomLayout extends BlockTemplateLayout
{
    protected $defaultOptions = [
        'columns' => 2,
        'gap' => 20,
        'class_name' => 'custom-layout',
    ];

    public function getHtmlStructure(): string
    {
        return '<div class="{{class_name}}">
            {{#items}}
                <div class="item">{{content}}</div>
            {{/items}}
        </div>';
    }
}
```

---

## Layout Registry

### Methods

| Method | Description |
|--------|-------------|
| `register(string $name, string $class)` | Register a new layout |
| `unregister(string $name)` | Remove a layout |
| `has(string $name)` | Check if layout exists |
| `get(string $name)` | Get layout class name |
| `resolve(string $name, array $options)` | Create layout instance |
| `all()` | Get all registered layouts |
| `getNames()` | Get all layout names |

### Example Usage

```php
// Get registry instance
$registry = LayoutRegistry::getInstance();

// List all layouts
$layouts = $registry->all();
foreach ($layouts as $name => $class) {
    echo "Layout: $name -> $class\n";
}

// Resolve with options
$grid = $registry->resolve('grid', [
    'columns' => 4,
    'gap' => 30,
    'responsive' => true,
]);
```

---

## Available Layouts

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

**Options:**
- `columns` (int): Number of columns (1-12)
- `gap` (int): Gap between items in pixels
- `responsive` (bool): Enable responsive behavior
- `class_name` (string): CSS class name

### List Layout

```php
use Jankx\Layouts\DynamicDataLayout\BlockLayouts\ListLayout;

$layout = new ListLayout();
$layout->setOptions([
    'style' => 'vertical',
    'class_name' => 'my-list',
]);
```

**Options:**
- `style` (string): 'vertical' | 'horizontal'
- `class_name` (string): CSS class name

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

**Options:**
- `slides_per_view` (int): Number of visible slides
- `autoplay` (bool): Enable autoplay
- `navigation` (bool): Show navigation arrows
- `pagination` (bool): Show pagination dots

---

## Creating Custom Layouts

### Step 1: Create Layout Class

```php
<?php
// includes/MyTheme/Layouts/CustomLayout.php

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

    public function getDefaultOptions(): array
    {
        return array_merge(parent::getDefaultOptions(), [
            'animation' => 'fade',
        ]);
    }
}
```

### Step 2: Register Layout

```php
// functions.php or service provider

use Jankx\Layouts\DynamicDataLayout\LayoutRegistry;
use MyTheme\Layouts\MasonryLayout;

add_action('jankx/layouts/register', function(LayoutRegistry $registry) {
    $registry->register('masonry', MasonryLayout::class);
});
```

### Step 3: Use in Blocks

```php
// In your Gutenberg block
$layout = LayoutRegistry::getInstance()->resolve('masonry', [
    'columns' => 4,
    'gap' => 20,
]);

return $layout->render($items);
```

---

## Usage Examples

### Dynamic Data Block

```php
use Jankx\Gutenberg\Blocks\DynamicDataLayoutBlock;

class MyCustomBlock extends DynamicDataLayoutBlock
{
    public function render($attributes, $content)
    {
        $layoutName = $attributes['layout'] ?? 'grid';
        $layout = LayoutRegistry::getInstance()->resolve($layoutName, [
            'columns' => $attributes['columns'] ?? 3,
        ]);

        $data = $this->fetchData($attributes);
        return $layout->render($data);
    }
}
```

### Template Integration

```php
// In your template file
$layout = LayoutRegistry::getInstance()->resolve('list', [
    'style' => 'vertical',
    'class_name' => 'post-list',
]);

$query = new WP_Query(['posts_per_page' => 10]);
$items = [];

while ($query->have_posts()) {
    $query->the_post();
    $items[] = [
        'title' => get_the_title(),
        'content' => get_the_excerpt(),
        'link' => get_permalink(),
    ];
}

echo $layout->render(['items' => $items]);
```

### React/Gutenberg Integration

```javascript
// Register layout selector in block editor
import { registerBlockType } from '@wordpress/blocks';
import { SelectControl } from '@wordpress/components';

registerBlockType('jankx/dynamic-data', {
    edit: ({ attributes, setAttributes }) => {
        return (
            <SelectControl
                label="Layout"
                value={attributes.layout}
                options={[
                    { label: 'Grid', value: 'grid' },
                    { label: 'List', value: 'list' },
                    { label: 'Carousel', value: 'carousel' },
                ]}
                onChange={(layout) => setAttributes({ layout })}
            />
        );
    },
});
```

---

## Best Practices

1. **Always use LayoutRegistry**: Don't instantiate layouts directly, use the registry for proper DI
2. **Validate options**: Use `setOptions()` with type checking
3. **Cache renders**: For expensive layouts, consider caching the output
4. **Responsive design**: Use the built-in responsive options
5. **Accessibility**: Include ARIA labels and proper semantic HTML

---

## API Reference

- [LayoutRegistry API](./api/layout-registry.md)
- [BlockTemplateLayout API](./api/block-template-layout.md)
- [Layout Interface](./api/layout-interface.md)
