# Why Jankx? What Makes It Different

> **Jankx is not just a WordPress theme. It's a modern theme framework that brings Laravel-style architecture, Drupal-style content management, and React-style component thinking to WordPress.**

## The Problem with Traditional WordPress Themes

Most WordPress themes fall into one of these categories:

- **Page Builder themes** (Divi, Elementor): Great for non-devs, terrible for developers. Bloated, slow, vendor lock-in.
- **Starter themes** (_s, Sage): Clean code, but you rebuild the same features repeatedly.
- **Framework themes** (Genesis): Powerful, but dated architecture, no Gutenberg integration.

**Jankx solves all of these.**

---

## 8 Unique Features You Won't Find Anywhere Else

### 1. Layout Registry System

**What it is:** A dynamic layout management system with Dependency Injection support.

**Why it matters:** Switch between Grid, List, Carousel, Masonry layouts **without changing template files**. Register new layouts via code or extensions.

```php
// Register a custom layout
$registry->register('masonry', MasonryLayout::class);

// Use it anywhere
$layout = $registry->resolve('grid', [
    'columns' => 3,
    'gap' => 20,
    'responsive' => true
]);
echo $layout->render($query);
```

**The competition:**
- Genesis: Hardcoded loops in template files
- Sage: Blade templates, no dynamic layout switching
- Astra: Only basic grid options via Customizer
- Divi: Shortcode-based, database-bloated

---

### 2. Extension System with Manifests

**What it is:** A full plugin-like extension system built into the theme. Extensions have lifecycle hooks, dependencies, and auto-activation.

**Why it matters:** Build modular, reusable features that can be shared between projects. Extensions can override core functionality from child themes.

```json
{
    "name": "SEO Enhancer",
    "id": "seo-enhancer",
    "version": "2.0.0",
    "dependencies": ["jankx-analytics"],
    "auto_activate": true,
    "requires": {
        "php": ">=7.4",
        "jankx": ">=2.0"
    }
}
```

**Lifecycle hooks:**
```php
class SeoExtension extends AbstractExtension {
    public function install()   { /* Create tables */ }
    public function activate()  { /* Register hooks */ }
    public function boot()      { /* Init services */ }
    public function deactivate(){ /* Cleanup */ }
    public function uninstall() { /* Remove tables */ }
}
```

**The competition:**
- Genesis: No extension system
- Sage: Composer packages, no lifecycle management
- Divi: Third-party plugins only

---

### 3. Laravel-Style DI Container

**What it is:** A full Dependency Injection Container with Service Providers, Singletons, and Auto-resolution.

**Why it matters:** Write clean, testable, loosely-coupled code. No more global functions or hardcoded dependencies.

```php
// Register a service
$app->singleton(ViewService::class, function ($app) {
    return new ViewService($app);
});

// Auto-injection
class GridLayout {
    public function __construct(ViewService $viewService) {
        // Automatically resolved by container
    }
}

// Facades for convenience
View::render('partials.card', $data);
Config::get('app.debug');
```

**The competition:**
- _s: No container, procedural code
- Genesis: Global functions everywhere
- Sage: None
- Astra: None

---

### 4. Deep Gutenberg Integration

**What it is:** Not just custom blocks, but a **Block Enhancement System** and advanced blocks other themes don't have.

**Custom blocks:**
- Swiper (Touch carousel with lazy loading)
- Modal (Popup system)
- SmartSearch (AJAX search with filters)
- IconPicker (Visual icon selector)
- Typography (Advanced text controls)
- AdvancedFilters (Faceted search)

**Block Extras (unique feature):**
```php
// Add responsive visibility to ANY block
add_filter('jankx/gutenberg/extra/responsive-visibility', function($extras) {
    return array_merge($extras, ['core/paragraph', 'core/heading']);
});

// Show/hide blocks based on device
// Desktop only: Show CTA buttons
// Mobile only: Show simplified menu
```

**Child Order control:** Reorder child blocks via UI without drag-drop.

**The competition:**
- Genesis: No Gutenberg integration
- Sage: Basic block support
- Astra: Only core blocks
- Divi: Blocks are shortcodes in disguise

---

### 5. JSON-Based Content Layout Patterns

**What it is:** Define complex layout patterns using JSON arrays instead of PHP templates.

**Why it matters:** Non-developers can modify layouts. Version-control friendly. Reusable across projects.

```php
// Card Overlay layout - defined as JSON
$layout = [
    ['core/cover', ['overlayColor' => 'black', 'dimRatio' => 50],
        [
            ['core/group', ['layout' => ['type' => 'flex']],
                [
                    ['core/post-title', ['level' => 3, 'isLink' => true]],
                    ['core/post-date', ['style' => ['color' => ['text' => '#eee']]]]
                ]
            ]
        ]
    ]
];
```

**Visual result:** Card with image background, dark overlay, title and date on top.

**The competition:**
- Genesis: PHP template functions
- Sage: Blade templates
- Divi: Database-stored layouts (lock-in!)

---

### 6. Mobile Detection & Adaptive Layouts

**What it is:** Server-side mobile detection that changes layout behavior, not just CSS.

**Why it matters:** Load different content for mobile vs desktop. Different columns, different components, optimized performance.

```php
class GridLayout {
    public function render() {
        if ($this->isMobile()) {
            // Mobile: Single column, simplified cards
            $this->columns = 1;
            $this->showExcerpt = false;
        }
        return parent::render();
    }
}
```

**The competition:**
- All others: CSS media queries only
- Jankx: Server-side + CSS = better performance

---

### 7. Font & Icon Management System (Pro)

**What it is:** Centralized font and icon management with visual pickers.

```php
// Add Google Fonts automatically
Fonts::add([
    'name' => 'Playfair Display',
    'family' => 'Playfair Display, serif',
    'category' => 'google'
]);

// Apply to elements
Fonts::apply('#site-title', 'Playfair Display');

// Icons with picker UI
Icons::render('fontawesome', 'star', ['class' => 'text-gold']);
```

**Features:**
- Automatic Google Fonts loading
- FontAwesome, Dashicons, Custom icons
- Admin UI for font selection
- Icon picker in Gutenberg

**The competition:**
- Genesis: Manual enqueue
- Sage: None
- Astra: Basic font options
- Divi: Has this, but bloated

---

### 8. AssetResolver with Anti-CLS

**What it is:** Smart CSS/JS management with Critical CSS injection and priority loading.

**Why it matters:** Eliminate Cumulative Layout Shift (CLS), improve Core Web Vitals, pass Google PageSpeed.

```php
// Add critical layout CSS inline (loads instantly)
$assetResolver->addInlineCss('
    .is-flex-container { display: flex; flex-wrap: wrap; gap: 1.5rem; }
    .has-image-ratio { position: relative; padding-top: 56.25%; }
', AssetResolver::CORE_LAYOUT);

// Async load non-critical CSS
$assetResolver->addStyle('swiper', $url, ['async' => true]);
```

**The competition:**
- All others: Basic `wp_enqueue_style`
- Jankx: Intelligent loading strategies

---

## Feature Comparison Matrix

| Feature | Jankx | Genesis | Sage | Astra | Divi |
|---------|-------|---------|------|-------|------|
| **DI Container** | ✅ Full | ❌ None | ❌ None | ❌ None | ❌ None |
| **Service Providers** | ✅ | ❌ | ❌ | ❌ | ❌ |
| **Facades** | ✅ | ❌ | ❌ | ❌ | ❌ |
| **Layout Registry** | ✅ Dynamic | ❌ Hardcoded | ❌ Static | ⚠️ Basic | ❌ Shortcodes |
| **Extension System** | ✅ w/ Manifest | ❌ | ❌ Composer only | ❌ | ❌ |
| **Gutenberg Blocks** | ✅ 15+ custom | ❌ | ⚠️ Basic | ❌ | ⚠️ Shortcodes |
| **Block Extras** | ✅ Unique | ❌ | ❌ | ❌ | ❌ |
| **Mobile Detection** | ✅ Server-side | ❌ | ❌ | ❌ | ❌ |
| **Font Manager** | ✅ Pro | ❌ | ❌ | ⚠️ Basic | ✅ |
| **Icon Manager** | ✅ Pro | ❌ | ❌ | ❌ | ✅ |
| **Critical CSS** | ✅ Built-in | ❌ | ❌ | ❌ | ❌ |
| **JSON Layouts** | ✅ Native | ❌ | ❌ | ❌ | ❌ Database |
| **Child Theme Overrides** | ✅ Extensions | ⚠️ Functions | ✅ Templates | ⚠️ Hooks | ❌ |
| **Testing Support** | ✅ PHPUnit | ❌ | ⚠️ | ❌ | ❌ |

---

## Real-World Use Cases

### Use Case 1: News/Magazine Site
**Challenge:** Multiple content types with different layouts (news grid, video carousel, list view).

**Jankx solution:**
```php
// Homepage
$registry->resolve('grid', ['columns' => 3])->render($news_query);

// Video section
$registry->resolve('carousel', ['slides_per_view' => 2])->render($video_query);

// Breaking news
$registry->resolve('list', ['style' => 'compact'])->render($breaking_query);
```

### Use Case 2: E-commerce with Custom Filters
**Challenge:** Advanced product filtering with faceted search.

**Jankx solution:**
- `AdvancedFilters` Gutenberg block
- Extension for WooCommerce integration
- Mobile-adaptive layout (filters collapse on mobile)

### Use Case 3: SaaS/Product Site
**Challenge:** Complex landing pages with modals, animations, icons.

**Jankx solution:**
- `Modal` block for pricing details
- `IconPicker` for feature lists
- `Swiper` for testimonials
- `Typography` for hero text

### Use Case 4: Multi-site Network
**Challenge:** Shared features across 50+ sites with customization per site.

**Jankx solution:**
- Core extensions in parent theme
- Site-specific extensions in child themes
- Shared layouts via LayoutRegistry

---

## Code Quality & Architecture

**Modern PHP:**
- PHP 7.4+ type declarations
- Namespacing (PSR-4)
- Strict typing where possible
- Composer autoloading

**Testing:**
- 266+ PHPUnit tests
- Mockery for mocking
- Code coverage tracking
- CI/CD ready

**Standards:**
- PSR-12 code style
- WordPress coding standards where applicable
- Semantic versioning

---

## Who Should Use Jankx?

**✅ Perfect for:**
- Developers building custom sites
- Agencies with multiple projects
- Teams needing maintainable code
- Performance-focused projects
- Headless/decoupled WordPress setups

**❌ Not for:**
- Non-technical users wanting drag-drop
- Simple brochure sites
- Teams committed to page builders

---

## Getting Started

```bash
# Install via Composer
composer require jankx/theme

# Or clone for development
git clone https://github.com/jankx/theme.git
```

**Documentation:** https://jankx.github.io/docs/

**GitHub:** https://github.com/jankx/theme

**Pro Version:** Contact for licensing

---

## The Bottom Line

If you're tired of:
- ❌ Bloated page builders
- ❌ Dated framework architecture  
- ❌ Rebuilding the same features
- ❌ Unmaintainable child theme functions.php

**Jankx is for you.**

It's the modern WordPress framework developers have been waiting for.

---

*Built with ❤️ by developers, for developers.*

*License: GPL-2.0+ (Lite) | Commercial (Pro)*
