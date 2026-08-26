# Registering a Web Font in Jankx Framework

Jankx includes a robust Fonts API that handles enqueueing, pre-connecting, and injecting fonts into Gutenberg (`theme.json`) dynamically.

## How to Register a Font in a Child Theme

1. **Create a Service Provider**
   In your child theme, create a new Service Provider (e.g. `src/Providers/FontsServiceProvider.php`) and register your font using the `Jankx\Facades\Fonts` facade.

   ```php
   <?php

   namespace Puleeno\Nibitour\Providers;

   use Jankx\Support\Providers\ServiceProvider;
   use Jankx\Foundation\Application;
   use Jankx\Facades\Fonts;

   class FontsServiceProvider extends ServiceProvider
   {
       public function register(Application $app)
       {
           // Use register() for container binding
       }

       public function boot(Application $app)
       {
           // Hook into Jankx font registration action
           add_action('jankx/fonts/register', [$this, 'registerFonts']);
       }

       public function registerFonts($fontsService)
       {
           // Example: Registering Google Font Be Vietnam Pro
           Fonts::google(
               'Be Vietnam Pro',
               ['100', '200', '300', '400', '500', '600', '700', '800', '900', '100i', '200i', '300i', '400i', '500i', '600i', '700i', '800i', '900i'],
               ['vietnamese', 'latin', 'latin-ext']
           );
       }
   }
   ```

2. **Register the Provider in Theme Config**
   Add your newly created Service Provider to your child theme's `config/app.php` file under the `providers` array:

   ```php
   return [
       // ...
       'providers' => [
           // Other providers...
           Puleeno\Nibitour\Providers\FontsServiceProvider::class,
       ],
       // ...
   ];
   ```

3. **Declare Font in `theme.json` (Child Theme)**
   To make the newly registered font accessible in the Gutenberg Editor, you should update the child theme's `theme.json` file. Override the `fontFamilies` settings and update the default body typography to use the newly registered font.

   ```json
   {
       "settings": {
           "typography": {
               "fontFamilies": [
                   {
                       "fontFamily": "\"Be Vietnam Pro\", -apple-system, BlinkMacSystemFont, \"Segoe UI\", Roboto, Oxygen, Ubuntu, Cantarell, \"Open Sans\", \"Helvetica Neue\", sans-serif",
                       "name": "Be Vietnam Pro",
                       "slug": "be-vietnam-pro"
                   },
                   {
                       "fontFamily": "\"Inter\", -apple-system, BlinkMacSystemFont, \"Segoe UI\", Roboto, Oxygen, Ubuntu, Cantarell, \"Open Sans\", \"Helvetica Neue\", sans-serif",
                       "name": "Inter",
                       "slug": "inter"
                   }
               ]
           }
       },
       "styles": {
           "typography": {
               "fontFamily": "var(--wp--preset--font-family--be-vietnam-pro)"
           }
       }
   }
   ```

## Other Font Registration Methods

The `Jankx\Facades\Fonts` facade provides various ways to register different types of fonts:

*   **Google Fonts**: `Fonts::google('Font Name', $variantsArray, $subsetsArray)`
*   **Adobe Fonts**: `Fonts::adobe('Font Name', 'projectId')`
*   **Custom Fonts**: `Fonts::custom('Font Name', 'font-family-css', 'path/to/font.css')`
*   **Custom from Local CSS**: `Fonts::customFromCss('Font Name', 'path/to/font.css')`

The Jankx `FontsService` will automatically handle generating the appropriate CSS requests, `<link rel="preconnect">` attributes (for Google/Adobe), and passing the registered entities to Gutenberg editors.
