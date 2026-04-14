# Contributing to Jankx

Thank you for your interest in contributing to Jankx! This guide will help you get started.

## Code of Conduct

- Be respectful and inclusive
- Provide constructive feedback
- Focus on what's best for the community

## How Can I Contribute?

### Reporting Bugs

Before creating a bug report:

1. Check if the issue already exists
2. Use the latest version to verify
3. Collect relevant information

**Bug report template:**
```
**Description:**
Clear description of the bug

**Steps to Reproduce:**
1. Go to '...'
2. Click on '...'
3. See error

**Expected Behavior:**
What should happen

**Environment:**
- WordPress version:
- PHP version:
- Browser:
```

### Suggesting Features

- Explain why this feature would be useful
- Provide examples of how it would work
- Consider the scope and complexity

### Pull Requests

1. Fork the repository
2. Create a branch (`git checkout -b feature/amazing-feature`)
3. Make your changes
4. Run tests (`./vendor/bin/phpunit`)
5. Commit (`git commit -m 'Add amazing feature'`)
6. Push (`git push origin feature/amazing-feature`)
7. Open a Pull Request

## Development Setup

```bash
# Clone your fork
git clone https://github.com/YOUR_USERNAME/theme.git

# Install dependencies
composer install
npm install

# Setup WordPress testing (requires local WP)
cd /path/to/wordpress/wp-content/themes/jankx

# Run tests
./vendor/bin/phpunit

# Run tests with coverage
./vendor/bin/phpunit --coverage-html coverage/
```

## Coding Standards

### PHP

- Follow PSR-12 coding standards
- Use strict typing where possible: `declare(strict_types=1);`
- Write docblocks for all methods
- Maximum line length: 120 characters

```php
<?php
declare(strict_types=1);

namespace Jankx\Features;

/**
 * Example class demonstrating coding standards
 */
class ExampleFeature
{
    /**
     * Process data and return formatted result
     *
     * @param array $data Input data
     * @param string $format Output format
     * @return string Formatted output
     */
    public function process(array $data, string $format): string
    {
        // Implementation
    }
}
```

### JavaScript

- Use ES6+ features
- Follow WordPress JavaScript coding standards
- Use ESLint before committing

```bash
npm run lint:js
npm run lint:css
```

### CSS/SCSS

- Use BEM naming convention
- Mobile-first approach
- CSS variables for theming

```scss
.jankx-component {
    &__element {
        // styles
    }
    
    &--modifier {
        // styles
    }
}
```

## Testing

### Writing Tests

All new features must include tests:

```php
<?php

namespace Tests\Features;

use Tests\Helpers\TestCase;
use Jankx\Features\ExampleFeature;

class ExampleFeatureTest extends TestCase
{
    public function testProcessReturnsString()
    {
        $feature = new ExampleFeature();
        $result = $feature->process(['key' => 'value'], 'json');
        
        $this->assertIsString($result);
    }
    
    public function testProcessWithInvalidData()
    {
        $feature = new ExampleFeature();
        
        $this->expectException(\InvalidArgumentException::class);
        $feature->process([], 'invalid');
    }
}
```

### Test Coverage

- Aim for 80%+ coverage for new code
- Critical paths must have 100% coverage
- Use Mockery for mocking

## Documentation

- Update relevant documentation for new features
- Add PHPDoc blocks to all public methods
- Include code examples where helpful

## Commit Message Guidelines

Use conventional commits format:

```
type(scope): subject

body (optional)

footer (optional)
```

**Types:**
- `feat`: New feature
- `fix`: Bug fix
- `docs`: Documentation only
- `style`: Code style (formatting, no logic change)
- `refactor`: Code refactoring
- `test`: Adding or updating tests
- `chore`: Maintenance tasks

**Examples:**
```
feat(layout): add masonry layout support

fix(admin): resolve notice display issue

docs: update contributing guide with testing info
```

## Extension Development

Want to create an extension? See the [Extensions Guide](../extensions/).

Quick start:
```php
namespace Jankx\Extensions\MyExtension;

use Jankx\Extensions\AbstractExtension;

class MyExtension extends AbstractExtension
{
    protected $id = 'my-extension';
    protected $name = 'My Extension';
    
    public function activate(): void
    {
        // Your code here
    }
}
```

## Questions?

- GitHub Discussions: https://github.com/jankx/jankx-lite/discussions
- Issues: https://github.com/jankx/jankx-lite/issues

## License

By contributing, you agree that your contributions will be licensed under the GPL-2.0+ License.
