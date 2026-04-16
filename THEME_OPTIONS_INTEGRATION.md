# 🎨 Theme Options Integration System

The Theme Options Integration System connects Theme Options, Gutenberg Blocks, and Global CSS into a unified, synchronized system. Changes in Theme Options automatically propagate to all blocks and CSS throughout the site.

![Pro](https://img.shields.io/badge/PRO-764ba2?style=flat-square)

---

## 🎯 Problem Solved

**Before:** Theme Options, Gutenberg Blocks, and Global CSS operated independently.
- User changes color in Theme Options → Blocks don't receive new color
- Container width in Theme Options different from theme.json layout
- Typography settings not applied to blocks

**After:** Fully synchronized system
- Change in Theme Options → Automatic CSS and Block updates
- CSS Variables connect all components

---

## 🏗️ Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                     THEME OPTIONS                           │
│  (primary_color, container_width, body_typography...)       │
└──────────────────────┬──────────────────────────────────────┘
                       │
                       ▼
┌─────────────────────────────────────────────────────────────┐
│          ThemeOptionsCSSGenerator                           │
│  Generates CSS Variables:                                   │
│  --jankx-primary-color, --jankx-container-width...          │
└──────────────────────┬──────────────────────────────────────┘
                       │
         ┌─────────────┴─────────────┐
         │                           │
         ▼                           ▼
┌────────────────────┐    ┌─────────────────────────────┐
│  GLOBAL CSS        │    │  ThemeOptionsBridge         │
│  (wp_add_inline_   │    │                             │
│   style)           │    │  • Pass data to JS          │
│                    │    │  • Filter theme.json         │
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

## 📂 System Components

| File | Purpose | When It Runs |
|------|---------|--------------|
| `includes/app/Services/ThemeOptionsCSSGenerator.php` | Generates CSS from theme options | Every page load |
| `includes/app/Services/ThemeOptionsBridge.php` | Connects to blocks and theme.json | Every page load |
| `includes/app/Providers/ThemeOptionsIntegrationServiceProvider.php` | Registers services | Theme boot |
| `includes/app/helpers/theme-options.php` | Helper functions | On demand |

---

## 🎨 CSS Variables Generated

When users change theme options, these CSS variables are generated:

```css
:root {
  /* Colors */
  --jankx-primary-color: #ff5722;      /* from primary_color */
  --jankx-secondary-color: #009688;   /* from secondary_color */
  --jankx-primary-color-rgb: 255, 87, 34;
  --jankx-secondary-color-rgb: 0, 150, 136;

  /* Layout */
  --jankx-container-width: 1200px;    /* from container_width */
  --jankx-sidebar-position: right;    /* from sidebar_position */

  /* Typography */
  --jankx-body-font-family: Inter;    /* from body_typography */
  --jankx-body-font-size: 16px;
  --jankx-body-font-weight: 400;
  --jankx-body-line-height: 1.6;
  --jankx-body-text-color: #222222;
}
```

---

## 💻 Developer Usage

### PHP Templates

```php
<?php
// Get theme option value
$primaryColor = jankx_get_theme_option('primary_color', '#ff5722');

// Get color in different formats
$hex = jankx_get_theme_color('primary', 'hex');        // #ff5722
$rgb = jankx_get_theme_color('primary', 'rgb');        // 255, 87, 34
$css = jankx_get_theme_color('primary', 'css-var');   // var(--jankx-primary-color)

// Get container width
$width = jankx_get_container_width();                  // 1200px
$widthValue = jankx_get_container_width(false);       // 1200 (number)

// Get typography
$font = jankx_get_body_typography('font-family');     // Inter
$allTypography = jankx_get_body_typography();         // Array of all values

// Get CSS variable
$var = jankx_get_css_var('primary-color');            // var(--jankx-primary-color)
$value = jankx_get_css_var('primary-color', true);     // #ff5722 (actual value)
?>

<!-- Usage in HTML -->
<div style="background-color: <?php echo esc_attr($css); ?>; max-width: <?php echo $width; ?>;">
    Content
</div>
```

### Block Rendering (PHP)

```php
<?php
namespace Jankx\Gutenberg\Blocks;

class MyBlock extends Block {

    public function render($attributes, $content, $block) {
        // Get theme colors
        $primaryColor = jankx_get_theme_color('primary', 'css-var');
        $containerWidth = jankx_get_css_var('container-width');

        // Create inline styles
        $styles = sprintf(
            'background-color: %s; max-width: %s;',
            $primaryColor,
            $containerWidth
        );

        // Or use CSS variables directly
        $styles = 'background-color: var(--jankx-primary-color);';

        return sprintf(
            '<div class="my-block" style="%s">%s</div>',
            esc_attr($styles),
            $content
        );
    }
}
```

### JavaScript (Block Editor)

```javascript
// Access theme options in block
const themeOptions = window.jankxThemeOptions;

// Data structure:
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

// Example: Block edit component
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

### SCSS/CSS

```scss
.my-block {
  // Use CSS variables
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

// Use RGB variables for opacity
.overlay {
  background-color: rgba(
    var(--jankx-primary-color-rgb),
    0.5  // 50% opacity
  );
}
```

### theme.json

Updated to use CSS variables as fallbacks:

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

## 👤 End User Usage (Non-technical)

### Change Colors

1. Go to **Appearance → Theme Options → Branding**
2. Change **Primary Color** and **Secondary Color**
3. Click **Save Changes**
4. **Automatic updates:**
   - Blocks using primary color will update
   - Link hovers change color
   - Theme palette syncs

### Change Container Width

1. Go to **Appearance → Theme Options → Layout**
2. Adjust **Container Width** (960px - 1440px)
3. Click **Save Changes**
4. **Automatic updates:**
   - Site-wide layout changes
   - Gutenberg editor updates too
   - Blocks maintain proportions

### Change Typography

1. Go to **Appearance → Theme Options → Typography**
2. Select **Body Font**
3. Change settings: Font family, Size, Weight, Line height
4. Click **Save Changes**
5. **Automatic updates:**
   - Site-wide font changes
   - Blocks using body font update
   - Editor also displays new font

### Change Sidebar Position

1. Go to **Appearance → Theme Options → Layout**
2. Select **Sidebar Position**: Right / Left / No Sidebar
3. Click **Save Changes**
4. **Automatic updates:**
   - Layout switches left/right/no sidebar
   - Body class changes (`jankx-sidebar-right`, `jankx-sidebar-left`, `jankx-sidebar-none`)
   - CSS responsive adjusts

---

## 🔧 Integrating New Blocks

When creating a new block, follow these steps:

### Step 1: PHP Block Class

```php
<?php
class NewBlock extends Block {
    protected $blockId = 'jankx/new-block';

    public function enqueueEditorAssets(): void {
        parent::enqueueEditorAssets();

        // Pass theme options to JS
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
        // Get theme colors
        $primary = jankx_get_theme_color('primary', 'css-var');
        $secondary = jankx_get_theme_color('secondary', 'css-var');

        // Create inline styles
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

### Step 2: block.json

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

### Step 3: edit.js (React)

```javascript
import { useSelect } from '@wordpress/data';

export default function Edit({ attributes, setAttributes }) {
    // Get theme options from window object
    const themeOptions = window.jankxNewBlockThemeOptions || {};
    const { useThemeColors } = attributes;

    // Or get from block editor settings
    const colors = useSelect((select) => {
        return select('core/block-editor').getSettings().colors;
    }, []);

    // Find primary color from palette
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

### Step 4: style.scss

```scss
.wp-block-jankx-new-block {
  // Use CSS variables from theme
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

## 🧪 Testing

### Using WP-CLI

```bash
# Run test script
wp eval-file wp-content/themes/jankx/tests/theme-options-integration-test.php
```

### Manual Testing

**In Admin:**
1. Open **Appearance → Theme Options**
2. Change **Primary Color** to different color (e.g., #ff0000)
3. Save changes
4. Open **Pages → Add New** (Block Editor)
5. Check:
   - Does primary color in color palette change to #ff0000?
   - Do blocks using primary color update?

**On Frontend:**
1. Open DevTools (F12)
2. Go to **Elements** tab
3. Find `<style id="jankx-theme-options-css">`
4. Verify CSS variables:
   ```css
   :root {
     --jankx-primary-color: #ff0000;  // Your changed color
   }
   ```

**In Console:**
```javascript
// Check theme options data
console.log(window.jankxThemeOptions);

// Expected result:
// {
//   colors: { primary: '#ff0000', secondary: '...' },
//   layout: { containerWidth: 1200, ... },
//   typography: { body: { ... } }
// }
```

---

## 🐛 Troubleshooting

### Issue 1: CSS Not Updating When Changing Theme Options

**Cause:** Cache

**Solution:**
```php
// In functions.php or custom plugin
add_action('jankx/options/updated', function() {
    // Clear CSS cache
    do_action('jankx/theme-options-integration/clear-cache');

    // Or use plugin cache clear
    if (function_exists('wp_cache_flush')) {
        wp_cache_flush();
    }
});
```

### Issue 2: Blocks Not Receiving Theme Colors

**Cause:** Block not integrated

**Solution:**
- Add `window.jankxThemeOptions` to block's editor script
- Use helper functions in block's render method

### Issue 3: theme.json Not Syncing

**Check:**
```php
// In functions.php
add_action('wp_theme_json_data_theme', function($theme_json) {
    error_log(print_r($theme_json->get_data(), true));
    return $theme_json;
});
```

---

## 📋 Integration Checklist for New Blocks

- [ ] Add `jankx_get_theme_options_data()` to `enqueueEditorAssets()`
- [ ] Use `jankx_get_theme_color()` in `render()` method
- [ ] Add CSS variables in `style.scss`
- [ ] Test with theme option changes
- [ ] Check responsive behavior
- [ ] Verify in editor and frontend

---

## 📚 Related Documentation

- **Theme Options Location:** `Appearance → Theme Options`
- **English Documentation:** `docs/theme-options-integration.md`
- **Vietnamese Guide:** `docs/theme-options-integration-huong-dan.md`
- **Test Script:** `tests/theme-options-integration-test.php`
- **Helper Functions:** `includes/app/helpers/theme-options.php`
