# Admin System

Jankx Theme's Admin System provides comprehensive tools for theme management, options handling, and form processing.

## Overview

```
┌─────────────────────────────────────────┐
│           Admin Panel                    │
│  ┌────────┐ ┌────────┐ ┌────────┐     │
│  │ Theme  │ │ Form   │ │ Admin  │     │
│  │Options │ │Handlers│ │Notices │     │
│  └────────┘ └────────┘ └────────┘     │
│         │           │          │        │
│         └───────────┴──────────┘        │
│              FormHandler                 │
└─────────────────────────────────────────┘
```

## Table of Contents

- [Form Handler](#form-handler)
- [Admin Notices](#admin-notices)
- [Theme Options](#theme-options)
- [Security](#security)
- [Best Practices](#best-practices)

---

## Form Handler

The `FormHandler` class processes admin form submissions with built-in security checks.

```php
use Jankx\Admin\Handlers\FormHandler;
use Jankx\Foundation\Application;

$app = Application::getInstance();
$formHandler = new FormHandler($app);

// Process requests (usually called on admin_init)
add_action('admin_init', [$formHandler, 'handleRequests']);
```

### Built-in Actions

| Action | Description | Capability | Version |
|--------|-------------|------------|:-------:|
| `save_image_sizes` | Save image size settings | `manage_options` | ![Lite](https://img.shields.io/badge/LITE-667eea?style=flat-square) ![Pro](https://img.shields.io/badge/PRO-764ba2?style=flat-square) |
| `clear_debug_log` | Clear debug log file | `manage_options` | ![Lite](https://img.shields.io/badge/LITE-667eea?style=flat-square) ![Pro](https://img.shields.io/badge/PRO-764ba2?style=flat-square) |
| `activate_license` | Activate license key | `manage_options` | ![Pro](https://img.shields.io/badge/PRO-764ba2?style=flat-square) |
| `deactivate_license` | Deactivate license | `manage_options` | ![Pro](https://img.shields.io/badge/PRO-764ba2?style=flat-square) |
| `disconnect_membership` | Disconnect membership | `manage_options` | ![Pro](https://img.shields.io/badge/PRO-764ba2?style=flat-square) |

### Custom Actions

```php
// Register custom action handler
add_action('jankx/form/handle/{action}', function($data, $formHandler) {
    // Handle custom action
    update_option('my_custom_option', $data['value']);
    
    // Add success notice
    $formHandler->addAdminNotice('Settings saved successfully!', 'success');
}, 10, 2);
```

### Example: Custom Form Handler

```php
<?php
// includes/Admin/Handlers/CustomFormHandler.php

namespace MyTheme\Admin\Handlers;

use Jankx\Admin\Handlers\FormHandler;

class CustomFormHandler extends FormHandler
{
    protected function handleSaveCustomSettings(array $data): void
    {
        // Verify nonce
        if (!$this->verifyNonce($data, 'custom_settings_nonce')) {
            $this->addAdminNotice('Security check failed', 'error');
            return;
        }
        
        // Check capabilities
        if (!current_user_can('manage_options')) {
            $this->addAdminNotice('Permission denied', 'error');
            return;
        }
        
        // Sanitize and save
        $settings = $this->sanitizeRequestData($data['settings'] ?? []);
        update_option('my_custom_settings', $settings);
        
        // Success notice
        $this->addAdminNotice('Custom settings saved!', 'success');
    }
}
```

---

## Admin Notices

Display feedback messages to admin users.

### Methods

```php
// Add a notice
$formHandler->addAdminNotice('Settings saved!', 'success');
$formHandler->addAdminNotice('Please check your input', 'warning');
$formHandler->addAdminNotice('An error occurred', 'error');
$formHandler->addAdminNotice('For your information', 'info');
```

### Notice Types

| Type | Color | Use Case |
|------|-------|----------|
| `success` | Green | Operation successful |
| `warning` | Yellow | Attention needed |
| `error` | Red | Operation failed |
| `info` | Blue | Informational |

### Custom Notice Renderer

```php
// Customize notice HTML
add_filter('jankx/admin/notice/html', function($html, $message, $type) {
    return sprintf(
        '<div class="my-custom-notice notice-%s">
            <span class="icon">%s</span>
            <span class="message">%s</span>
        </div>',
        $type,
        $this->getIconForType($type),
        esc_html($message)
    );
}, 10, 3);
```

---

## Theme Options

### Service Provider Setup

```php
// includes/App/Providers/ThemeOptionsServiceProvider.php

namespace App\Providers;

use Jankx\Foundation\Providers\ServiceProvider;
use App\Services\ThemeOptionsService;

class ThemeOptionsServiceProvider extends ServiceProvider
{
    public function register(): void
    {
        $this->app->singleton('theme-options', function($app) {
            return new ThemeOptionsService($app);
        });
    }
    
    public function boot(): void
    {
        add_action('after_setup_theme', [$this->app->get('theme-options'), 'init']);
    }
}
```

### Options Service

```php
<?php
// includes/App/Services/ThemeOptionsService.php

namespace App\Services;

use Jankx\Foundation\Application;
use Jankx\Facades\Config;

class ThemeOptionsService
{
    protected $app;
    protected $options = [];
    
    public function __construct(Application $app)
    {
        $this->app = $app;
        $this->loadOptions();
    }
    
    public function init(): void
    {
        add_action('admin_menu', [$this, 'addAdminMenu']);
        add_action('admin_init', [$this, 'registerSettings']);
    }
    
    public function addAdminMenu(): void
    {
        add_theme_page(
            'Theme Options',
            'Theme Options',
            'manage_options',
            'theme-options',
            [$this, 'renderOptionsPage']
        );
    }
    
    public function registerSettings(): void
    {
        register_setting('jankx_theme_options', 'jankx_options', [
            'sanitize_callback' => [$this, 'sanitizeOptions'],
        ]);
    }
    
    public function get(string $key, $default = null)
    {
        return $this->options[$key] ?? $default;
    }
    
    public function set(string $key, $value): void
    {
        $this->options[$key] = $value;
        update_option('jankx_options', $this->options);
    }
    
    private function loadOptions(): void
    {
        $this->options = get_option('jankx_options', []);
    }
    
    public function sanitizeOptions(array $input): array
    {
        $sanitized = [];
        
        foreach ($input as $key => $value) {
            $sanitized[$key] = $this->sanitizeField($key, $value);
        }
        
        return $sanitized;
    }
    
    private function sanitizeField(string $key, $value)
    {
        switch ($key) {
            case 'google_analytics':
                return sanitize_text_field($value);
            case 'custom_css':
                return wp_kses_post($value);
            default:
                return sanitize_text_field($value);
        }
    }
}
```

### Options Page Template

```php
// includes/App/Views/admin/options-page.php

<div class="wrap">
    <h1><?php echo esc_html(get_admin_page_title()); ?></h1>
    
    <form method="post" action="options.php">
        <?php settings_fields('jankx_theme_options'); ?>
        
        <table class="form-table">
            <tr>
                <th scope="row">Google Analytics ID</th>
                <td>
                    <input type="text" 
                           name="jankx_options[google_analytics]" 
                           value="<?php echo esc_attr($options['google_analytics'] ?? ''); ?>"
                           class="regular-text">
                </td>
            </tr>
            <tr>
                <th scope="row">Custom CSS</th>
                <td>
                    <textarea name="jankx_options[custom_css]" 
                              rows="10" 
                              class="large-text code"><?php 
                        echo esc_textarea($options['custom_css'] ?? ''); 
                    ?></textarea>
                </td>
            </tr>
        </table>
        
        <?php submit_button(); ?>
    </form>
</div>
```

### Font Manager ![Pro](https://img.shields.io/badge/PRO-764ba2?style=flat-square)

Manage custom fonts through the admin panel:

```php
use Jankx\Facades\Fonts;

// Add custom font
Fonts::add([
    'name' => 'Custom Font',
    'family' => 'Custom, sans-serif',
    'category' => 'custom',
]);

// Get active fonts
$fonts = Fonts::getActive();

// Apply font to element
Fonts::apply('#header', 'Custom Font');
```

**Features:**
- Google Fonts integration
- Custom font upload
- Font pairing suggestions
- Live preview

### Icon Manager ![Pro](https://img.shields.io/badge/PRO-764ba2?style=flat-square)

Manage icon libraries:

```php
use Jankx\Facades\Icons;

// Get icons by type
$icons = Icons::get('fontawesome');

// Get all available icon sets
$sets = Icons::getSets();

// Render icon
Icons::render('star', ['class' => 'text-primary']);
```

**Supported Icon Sets:**
- FontAwesome 6
- Material Icons
- Bootstrap Icons
- Custom SVG icons

---

## Security

### Nonce Verification

```php
// Generate nonce
$nonce = wp_create_nonce('my_action');

// Verify in handler
if (!wp_verify_nonce($_POST['_wpnonce'] ?? '', 'my_action')) {
    wp_die('Security check failed');
}
```

### Capability Checks

```php
// Check user capabilities
if (!current_user_can('manage_options')) {
    wp_die('You do not have permission');
}

// Custom capability
current_user_can('jankx_edit_advanced_options');
```

### Data Sanitization

```php
// FormHandler provides built-in sanitization
$data = $this->sanitizeRequestData($_POST);

// Manual sanitization
$clean = [
    'text' => sanitize_text_field($input['text']),
    'email' => sanitize_email($input['email']),
    'url' => esc_url_raw($input['url']),
    'html' => wp_kses_post($input['html']),
    'key' => sanitize_key($input['key']),
];
```

### SQL Security

```php
// Use $wpdb with prepare
global $wpdb;
$results = $wpdb->get_results($wpdb->prepare(
    "SELECT * FROM {$wpdb->prefix}my_table WHERE id = %d",
    $id
));
```

---

## Best Practices

### 1. Always Verify Nonces

```php
// Good
if (!wp_verify_nonce($_POST['nonce'], 'my_action')) {
    wp_die('Invalid nonce');
}

// Bad
// No nonce verification!
update_option('setting', $_POST['value']);
```

### 2. Check Capabilities

```php
// Good
if (!current_user_can('manage_options')) {
    return;
}

// Bad
if (is_admin()) {
    update_option('setting', $value);
}
```

### 3. Validate Input

```php
// Good
$value = intval($_POST['number'] ?? 0);
if ($value < 0 || $value > 100) {
    $value = 0;
}

// Bad
$value = $_POST['number'];
```

### 4. Escape Output

```php
// Good
echo esc_html($title);
echo esc_url($url);
echo esc_attr($value);

// Bad
echo $title;
```

### 5. Use Admin Notices Properly

```php
// Good
$formHandler->addAdminNotice(
    __('Settings saved successfully.', 'jankx'),
    'success'
);

// Bad - direct echo
echo '<div class="notice">Done</div>';
```

---

## Complete Example: Admin Page

```php
<?php
/**
 * Admin Page Example
 */

namespace MyTheme\Admin;

use Jankx\Admin\Handlers\FormHandler;
use Jankx\Foundation\Application;

class MyAdminPage
{
    protected $app;
    protected $formHandler;
    
    public function __construct(Application $app)
    {
        $this->app = $app;
        $this->formHandler = new FormHandler($app);
    }
    
    public function init(): void
    {
        add_action('admin_menu', [$this, 'addMenuPage']);
        add_action('admin_init', [$this->formHandler, 'handleRequests']);
        
        // Register custom action
        add_action('jankx/form/handle/save_my_settings', [$this, 'saveSettings'], 10, 2);
    }
    
    public function addMenuPage(): void
    {
        add_menu_page(
            'My Settings',
            'My Settings',
            'manage_options',
            'my-settings',
            [$this, 'renderPage'],
            'dashicons-admin-generic',
            30
        );
    }
    
    public function renderPage(): void
    {
        $settings = get_option('my_settings', []);
        ?>
        <div class="wrap">
            <h1><?php echo esc_html(get_admin_page_title()); ?></h1>
            
            <form method="post">
                <?php wp_nonce_field('save_my_settings'); ?>
                <input type="hidden" name="jankx_action" value="save_my_settings">
                
                <table class="form-table">
                    <tr>
                        <th>Setting Name</th>
                        <td>
                            <input type="text" 
                                   name="setting_name" 
                                   value="<?php echo esc_attr($settings['name'] ?? ''); ?>"
                                   class="regular-text">
                        </td>
                    </tr>
                </table>
                
                <?php submit_button('Save Settings'); ?>
            </form>
        </div>
        <?php
    }
    
    public function saveSettings(array $data, FormHandler $handler): void
    {
        $settings = [
            'name' => sanitize_text_field($data['setting_name'] ?? ''),
        ];
        
        update_option('my_settings', $settings);
        
        $handler->addAdminNotice(
            __('Settings saved!', 'jankx'),
            'success'
        );
    }
}

// Initialize
add_action('admin_init', function() {
    $app = Application::getInstance();
    $page = new MyAdminPage($app);
    $page->init();
});
```

---

## API Reference

- [FormHandler API](./api/form-handler.md)
- [Admin Notices API](./api/admin-notices.md)
- [ThemeOptionsService API](./api/theme-options-service.md)
