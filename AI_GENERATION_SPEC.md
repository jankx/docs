# Jankx AI Generation Specification

This document provides technical mapping rules for AI models to generate Jankx-compliant components and themes from visual designs (Figma/Images).

## 1. CSS Level Mapping Rules

When generating CSS, categorize rules into the following levels:

### CORE_LAYOUT (Level 1)
- **What to include**: Base layout tokens, utility classes, and fundamental design rules.
- **Examples**: 
    - Flexbox containers (`.is-flex-container`)
    - Image Aspect Ratio foundations (`.has-image-ratio`)
    - Generic Grid Column systems (`.columns-3`)
- **Action**: Add to `ContentLayoutServiceProvider::boot()` via `AssetResolver::CORE_LAYOUT`.

### LAYOUT_TYPE (Level 2)
- **What to include**: Styles shared by multiple instances of the same component type.
- **Examples**:
    - Carousel navigation arrows and dots.
    - Post card skins (e.g. "Card Overlay", "Simple Card").
    - Taxonomy badge styles.
- **Action**: Add to the `Parser` class of the layout via `AssetResolver::LAYOUT_TYPE`.

### INSTANCE (Level 3)
- **What to include**: Styles unique to a specific block instance, often involving instance-specific IDs.
- **Examples**:
    - Specific image ratio for a single block (`#jkx-layout-123 { --jankx-image-ratio: 75%; }`).
    - Custom colors/margins applied only to one block instance.
- **Action**: Add to the `generateDynamicCss()` method in the `Parser` class.

## 2. Component Logic Mapping

Each UI component in a design should map to:
1. **PHP Block Class**: `includes/framework/Gutenberg/Blocks/{Name}Block.php`
2. **Data Parser**: `includes/framework/Layouts/DynamicDataLayout/Parsers/{Name}LayoutDataParser.php`
3. **Template**: `includes/framework/Layouts/templates/post-layout/{name}.php`

### AI Instruction for Templates:
Always use Semantic HTML. Use classes defined in `CORE_LAYOUT` or `LAYOUT_TYPE` instead of inline styles.

- **Bad**: `<div style="display: flex; gap: 10px;">`
- **Good**: `<div class="is-flex-container">`

## 3. SEO & Performance Requirements

1. **Anti-CLS**: Always provide aspect ratio CSS using the `--jankx-image-ratio` variable.
2. **Deduplication**: Never repeat the same CSS block. If multiple components share code, move it to `LAYOUT_TYPE`.
3. **Lazy Loading**: If a component requires a library (like Swiper or Embla), only load it when the component is active.
4. **Unique IDs**: Use `AssetResolver::generateUniqueId()` for elements needing specific scoping.

## 5. Block Logic Generation Rules

When generating the PHP Block Class (`includes/framework/Gutenberg/Blocks/{Name}Block.php`):

1. **Namespace**: Always use `namespace Jankx\Gutenberg\Blocks;`.
2. **Inheritance**: Extend `Jankx\Gutenberg\Block`.
3. **Property `$blockId`**: Must match the `name` in `block.json` (e.g., `jankx/my-block`).
4. **Render Method**:
    - Use `\extract()` for attributes.
    - Call global WP functions with leading backslash `\`.
    - Use `\Jankx\Facades\Asset::CORE_LAYOUT` and `LAYOUT_TYPE` for CSS injection.
    - Logic for fetching data should ideally be moved to a Service or a Data Parser if complex.

## 6. Vibe (TypeScript) to Jankx Mapping

| Vibe Construct (TS) | Jankx Construct (PHP/WP) | Location |
|---------------------|--------------------------|----------|
| Component Props     | Block Attributes         | `block.json` -> `"attributes"` |
| `useEffect` (Data)   | Service/Data Parser      | `includes/framework/Layouts/DynamicDataLayout/Parsers/` |
| JSX Template        | Plates/PHP Template      | `includes/framework/Layouts/templates/post-layout/` |
| Component Styles    | SCSS/Dynamic CSS         | `resources/blocks/{name}/style.scss` |
| Component State     | Block Attributes/JS      | `resources/blocks/{name}/index.tsx` |

## 7. Ground Truth Examples

Refer to:
- `Jankx\Services\AssetResolver` for CSS management logic.
- `Jankx\Layouts\DynamicDataLayout\Parsers\CarouselLayoutDataParser` for level 2 CSS usage.
- `Jankx\Gutenberg\Blocks\AuthorBoxBlock` for a complete block implementation.
- `Jankx\Layouts\templates\post-layout\carousel.php` for clean template implementation.

## 8. Absolute Layout Parity Rule

To ensure the output is **absolutely accurate** and identical between the Gutenberg Editor and the Frontend:

1. **DOM Skeleton Sync**: The HTML structure (tags, nesting) in `index.tsx` (Editor) and `Block.php` (Frontend) must be **identical**.
2. **Class Consistency**: Every class added to the frontend wrapper via `\get_block_wrapper_attributes()` must also be present in the editor via `useBlockProps()`.
3. **No Dynamic Logic in HTML**: Avoid logic like `if ($something) echo '<div>'` inside templates. Use CSS classes to hide/show elements instead, so the DOM skeleton remains consistent.

## 9. CSS Variable Bridging

Instead of complex style calculations in both JS and PHP, use CSS Variables:

- **Logic**: Attributes (like `gap`, `columns`, `primaryColor`) are mapped to CSS Variables (e.g., `--jankx-gap`, `--jankx-columns`).
- **PHP**: Inject variables into the style attribute of the wrapper via `AssetResolver::INSTANCE`.
- **JS**: Pass variables into the `style` prop of `useBlockProps`.
- **CSS**: The block-level `.scss` uses these variables for layout, ensuring both environments render exactly the same layout without duplicated logic.
