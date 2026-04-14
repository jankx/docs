# Extension System

Jankx Theme's Extension System allows you to modularize functionality and load features dynamically.

## Overview

```
┌─────────────────────────────────────────┐
│         ExtensionManager                │
│  ┌─────────┐ ┌─────────┐ ┌─────────┐  │
│  │ Plugin  │ │ Theme   │ │ Custom  │  │
│  │ Ext A   │ │ Ext B   │ │ Ext C   │  │
│  └────┬────┘ └────┬────┘ └────┬────┘  │
│       └────────────┴────────────┘       │
│              Manifest                    │
└─────────────────────────────────────────┘
```

## Table of Contents

- [Core Concepts](#core-concepts)
- [Extension Manager](#extension-manager)
- [Creating Extensions](#creating-extensions)
- [Extension Manifest](#extension-manifest)
- [Lifecycle Hooks](#lifecycle-hooks)
- [Best Practices](#best-practices)

---

## Core Concepts

### ExtensionManager

The central hub for managing all extensions:

```php
use Jankx\Extensions\ExtensionManager;

// Get singleton instance
$manager = ExtensionManager::getInstance();

// Check if extension is active
if ($manager->is_extension_active('my-extension')) {
    // Extension is active
}

// Get all extensions
$extensions = $manager->get_extensions();

// Get specific extension
$extension = $manager->get_extension('my-extension');
```

### Extension Types

| Type | Description | Location |
|------|-------------|----------|
| **Core** | Built-in extensions | `includes/extensions/` |
| **Theme** | Theme-specific | `includes/app/Extensions/` |
| **Plugin** | Third-party | Via plugin API |

---

## Extension Manager

### API Reference

```php
// Activation
$manager->activate_extension('my-extension');
$manager->deactivate_extension('my-extension');

// Installation
$manager->install_extension('my-extension');
$manager->uninstall_extension('my-extension');

// Requirements
$manager->require_extension('required-ext', true);   // Required
$manager->require_extension('optional-ext', false);  // Recommended

// Queries
$manager->is_extension_active('my-extension');
$manager->is_extension_required('my-extension');
$manager->is_extension_installed('my-extension');
```

### Events

```php
// Fired when extension is activated
do_action('jankx/extension/activated', $extensionName);

// Fired when extension is deactivated
do_action('jankx/extension/deactivated', $extensionName);

// Filter extension list
$extensions = apply_filters('jankx/extensions/list', $extensions);
```

---

## Creating Extensions

### Step 1: Create Extension Class

```php
<?php
// includes/extensions/my-extension/MyExtension.php

namespace Jankx\Extensions\MyExtension;

use Jankx\Extensions\AbstractExtension;

class MyExtension extends AbstractExtension
{
    protected $id = 'my-extension';
    protected $name = 'My Extension';
    protected $version = '1.0.0';
    protected $author = 'Your Name';
    protected $description = 'Extension description';
    
    public function activate(): void
    {
        parent::activate();
        
        // Activation logic
        $this->register_post_types();
        $this->register_taxonomies();
    }
    
    public function deactivate(): void
    {
        // Cleanup logic
        
        parent::deactivate();
    }
    
    public function boot(): void
    {
        // Run after all extensions loaded
        add_action('init', [$this, 'init']);
    }
    
    public function init(): void
    {
        // WordPress init hook
    }
    
    private function register_post_types(): void
    {
        register_post_type('my_cpt', [
            'public' => true,
            'label' => 'My Custom Post Type',
        ]);
    }
}
```

### Step 2: Create Manifest

```json
{
    "name": "My Extension",
    "id": "my-extension",
    "version": "1.0.0",
    "author": "Your Name",
    "description": "Extension description",
    "main": "MyExtension.php",
    "class": "Jankx\\Extensions\\MyExtension\\MyExtension",
    "requires": {
        "php": ">=7.4",
        "wordpress": ">=5.8",
        "jankx": ">=2.0"
    },
    "dependencies": [
        "other-extension"
    ],
    "hooks": {
        "activate": "on_activate",
        "deactivate": "on_deactivate"
    }
}
```

### Step 3: Register Extension

```php
// includes/extensions/my-extension/init.php

use Jankx\Extensions\ExtensionManager;
use Jankx\Extensions\MyExtension\MyExtension;

add_action('jankx/extensions/init', function(ExtensionManager $manager) {
    $manager->add_extension('my-extension', new MyExtension());
});
```

---

## Extension Manifest

### Manifest Schema

```json
{
    "$schema": "https://jankx.io/schemas/extension-manifest.json",
    "name": "string",
    "id": "string",
    "version": "string",
    "author": "string",
    "description": "string",
    "main": "string",
    "class": "string",
    "icon": "string",
    "screenshot": "string",
    "tags": ["string"],
    "category": "string",
    "requires": {
        "php": "string",
        "wordpress": "string",
        "jankx": "string",
        "extensions": ["string"]
    },
    "supports": {
        "gutenberg": true,
        "classic": true
    },
    "settings": {
        "page": "string",
        "section": "string"
    },
    "i18n": {
        "domain": "string",
        "path": "string"
    }
}
```

### Loading Manifest

```php
// ExtensionManager automatically loads manifest.json
$manager->load_extension_from_directory('/path/to/extension');

// Or manually
$manifest = json_decode(file_get_contents('manifest.json'), true);
$manager->register_from_manifest($manifest);
```

---

## Lifecycle Hooks

### Extension Lifecycle

```
┌─────────┐    ┌──────────┐    ┌─────────┐    ┌────────┐
│ Install │ -> │ Activate │ -> │   Boot  │ -> │  Init  │
└─────────┘    └──────────┘    └─────────┘    └────────┘
                                    │
                                    v
                             ┌──────────┐
                             │ Deactivate│
                             └──────────┘
```

### Available Methods

```php
class MyExtension extends AbstractExtension
{
    // Called on installation
    public function install(): bool
    {
        // Create tables, options, etc.
        return true;
    }
    
    // Called on activation
    public function activate(): void
    {
        // Register hooks, flush rewrite rules
    }
    
    // Called after all extensions loaded
    public function boot(): void
    {
        // Initialize services
    }
    
    // Called on WordPress init
    public function init(): void
    {
        // Register shortcodes, widgets
    }
    
    // Called on deactivation
    public function deactivate(): void
    {
        // Cleanup, flush cache
    }
    
    // Called on uninstall
    public function uninstall(): bool
    {
        // Remove tables, options
        return true;
    }
}
```

### WordPress Integration

```php
class MyExtension extends AbstractExtension
{
    public function boot(): void
    {
        // Register assets
        add_action('wp_enqueue_scripts', [$this, 'enqueue_assets']);
        
        // Register blocks
        add_action('init', [$this, 'register_blocks']);
        
        // Admin menu
        add_action('admin_menu', [$this, 'add_admin_menu']);
    }
    
    public function enqueue_assets(): void
    {
        wp_enqueue_style(
            'my-extension',
            $this->get_url() . '/assets/css/style.css',
            [],
            $this->version
        );
    }
}
```

---

## Best Practices

### 1. Namespace Properly

```php
// Good
namespace Jankx\Extensions\MyExtension;

// Bad
namespace MyExtension;
```

### 2. Use Dependency Injection

```php
class MyExtension extends AbstractExtension
{
    protected $repository;
    
    public function __construct(DataRepository $repository)
    {
        $this->repository = $repository;
    }
}

// Register with DI
$manager->add_extension('my-extension', $app->make(MyExtension::class));
```

### 3. Handle Dependencies

```php
public function boot(): void
{
    // Check dependencies
    if (!$this->manager->is_extension_active('required-ext')) {
        add_action('admin_notices', function() {
            echo '<div class="error">My Extension requires Required Extension</div>';
        });
        return;
    }
    
    // Continue initialization
}
```

### 4. Version Management

```php
class MyExtension extends AbstractExtension
{
    public function activate(): void
    {
        $current_version = get_option('my_extension_version', '0.0.0');
        
        if (version_compare($current_version, $this->version, '<')) {
            $this->run_migrations($current_version);
            update_option('my_extension_version', $this->version);
        }
        
        parent::activate();
    }
    
    private function run_migrations(string $from_version): void
    {
        // Database migrations
    }
}
```

### 5. Error Handling

```php
use Jankx\Facades\Log;

public function boot(): void
{
    try {
        $this->initialize_services();
    } catch (\Exception $e) {
        Log::error('MyExtension: Boot failed - ' . $e->getMessage());
        
        add_action('admin_notices', function() use ($e) {
            echo '<div class="error">My Extension failed to load</div>';
        });
    }
}
```

---

## Pro Features

### License Management ![Pro](https://img.shields.io/badge/PRO-764ba2?style=flat-square)

Protect your premium extensions with license validation:

```php
use Jankx\Facades\License;

// Check license
if (License::is_valid()) {
    // License is valid
}

// Activate license
License::activate('your-license-key');

// Get license info
$info = License::get_info();
```

### Auto-Updates ![Pro](https://img.shields.io/badge/PRO-764ba2?style=flat-square)

Enable automatic updates for extensions:

```php
// Enable auto-update for specific extension
add_filter('jankx/extension/auto_update', function($enabled, $extension) {
    if ($extension === 'my-premium-ext') {
        return true;
    }
    return $enabled;
}, 10, 2);
```

---

## Example: Complete Extension

```php
<?php
/**
 * Plugin Name: Jankx SEO Extension
 * Description: SEO optimization for Jankx Theme
 * Version: 1.0.0
 * Author: Your Name
 */

namespace Jankx\Extensions\Seo;

use Jankx\Extensions\AbstractExtension;
use Jankx\Facades\Config;

class SeoExtension extends AbstractExtension
{
    protected $id = 'seo';
    protected $name = 'SEO Extension';
    protected $version = '1.0.0';
    protected $author = 'Your Name';
    
    public function boot(): void
    {
        // Meta tags
        add_action('wp_head', [$this, 'output_meta_tags']);
        
        // Schema markup
        add_filter('jankx/schema_markup', [$this, 'add_schema_markup']);
        
        // Sitemap
        add_action('init', [$this, 'register_sitemap']);
    }
    
    public function output_meta_tags(): void
    {
        if (is_singular()) {
            echo '<meta name="description" content="' . 
                 esc_attr($this->get_meta_description()) . '">';
        }
    }
    
    private function get_meta_description(): string
    {
        if (is_singular()) {
            return get_post_meta(get_the_ID(), '_seo_description', true) 
                ?: get_the_excerpt();
        }
        return get_bloginfo('description');
    }
    
    public function add_schema_markup(array $schema): array
    {
        $schema['@context'] = 'https://schema.org';
        $schema['@type'] = is_singular() ? 'Article' : 'WebPage';
        
        return $schema;
    }
}

// Register
add_action('jankx/extensions/init', function($manager) {
    $manager->add_extension('seo', new SeoExtension());
});
```

---

## API Reference

- [AbstractExtension API](./api/abstract-extension.md)
- [ExtensionManager API](./api/extension-manager.md)
- [ExtensionManifest API](./api/extension-manifest.md)
