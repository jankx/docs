# Why Jankx? What Makes It Different

> **Jankx is a modern WordPress theme framework that brings Laravel-style architecture to developers today, with a vision of becoming friendly to end users tomorrow.**

## Different Approaches, Different Strengths

WordPress themes serve different needs. Each approach has its place:

- **Page Builder themes** (Divi, Elementor): Excellent for visual building, drag-and-drop convenience. Best for users who prefer GUI over code.
- **Starter themes** (_s, Sage): Clean foundations for custom development. Great starting points that require building features from scratch.
- **Framework themes** (Genesis): Proven architecture with extensive hooks. Established patterns that prioritize stability.

**Jankx takes a different path** - bringing modern PHP practices (DI containers, modular architecture) into WordPress theme development, with deep Gutenberg integration.

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

**Other approaches:**
- Genesis: Template-based layouts with hooks
- Sage: Blade templating system
- Astra: Customizer-based layout options
- Divi: Visual builder with shortcode storage

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

**Other approaches:**
- Genesis: Child themes with functions.php hooks
- Sage: Composer-based dependencies
- Divi: Third-party plugin ecosystem

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

**Other approaches:**
- _s: Standard WordPress procedural patterns
- Genesis: Global function-based architecture
- Sage: Plain PHP without DI container

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

**Other approaches:**
- Genesis: Primarily classic editor focused
- Sage: Basic Gutenberg support
- Astra: Core Gutenberg blocks
- Divi: Visual builder blocks

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

**Other approaches:**
- Genesis: PHP template functions
- Sage: Blade templates
- Divi: Database-stored layouts

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

**Other approaches:**
- Genesis: Manual enqueue functions
- Sage: Standard WordPress enqueue
- Astra: Customizer font options
- Divi: Built-in font management

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

## What Jankx Brings to the Table

Here's how Jankx approaches common development needs:

| Approach | Jankx | Other Themes |
|----------|-------|--------------|
| **Architecture** | DI Container + Service Providers (Laravel-inspired) | Hooks/filters (Genesis), Plain PHP (_s), Blade (Sage) |
| **Layouts** | Registry pattern - switch layouts dynamically via code | Template files, Customizer options, or builder interface |
| **Extensibility** | Built-in Extension system with lifecycle management | Child themes, plugins, or composer packages |
| **Gutenberg** | Custom blocks + Block Extras (responsive visibility, etc.) | Core blocks or third-party page builders |
| **Mobile** | Server-side detection + responsive CSS | CSS media queries primarily |
| **Testing** | 266+ PHPUnit tests included | Testing setup left to developer |

*Each tool has its strengths. Jankx is designed for developers who prefer framework-style architecture and want modern PHP patterns in their WordPress themes.*

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

**Currently best for:**
- Developers building custom sites
- Agencies with multiple projects
- Teams needing maintainable code
- Performance-focused projects
- Headless/decoupled WordPress setups

**Our Vision:** While Jankx currently requires code knowledge, our ultimate goal is to become friendly to end users too - bringing the power of modern framework architecture into an interface anyone can use.

---

## Getting Started

```bash
# Install via Composer
composer require jankx/jankx

# Or clone for development
git clone https://github.com/jankx/jankx-lite.git
```

**Documentation:** https://jankx.github.io/docs/

**GitHub:** https://github.com/jankx/jankx-lite

**Pro Version:** Contact for licensing

---

## The Bottom Line

Jankx offers a fresh approach to WordPress theme development - combining modern PHP architecture with deep WordPress integration.

Whether you're building a complex custom site or looking for a framework that grows with your team, Jankx provides the foundation.

---

*Built with ❤️ by developers, for developers.*

*License: GPL-2.0+ (Lite) | Commercial (Pro)*
