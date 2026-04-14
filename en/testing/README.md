# Testing Guide

Comprehensive testing documentation for Jankx Theme.

## Overview

```
Test Coverage
├── Unit Tests (266 tests)
│   ├── Admin Tests
│   ├── Extension Tests
│   ├── Layout Tests
│   ├── Service Tests
│   └── Provider Tests
├── Integration Tests
└── Code Coverage Reports
```

## Table of Contents

- [Getting Started](#getting-started)
- [Test Structure](#test-structure)
- [Running Tests](#running-tests)
- [Writing Tests](#writing-tests)
- [Mocking WordPress](#mocking-wordpress)
- [Code Coverage](#code-coverage)
- [Best Practices](#best-practices)

---

## Getting Started

### Installation

```bash
# Install PHPUnit and dependencies
composer install --dev

# Verify installation
./vendor/bin/phpunit --version
```

### Configuration

`phpunit.xml`:
```xml
<phpunit bootstrap="tests/bootstrap.php"
         colors="true"
         verbose="true">
    <testsuites>
        <testsuite name="Admin">
            <directory>tests/Admin</directory>
        </testsuite>
        <testsuite name="Extensions">
            <directory>tests/Extensions</directory>
        </testsuite>
        <testsuite name="Layouts">
            <directory>tests/Layouts</directory>
        </testsuite>
        <testsuite name="Services">
            <directory>tests/Services</directory>
        </testsuite>
    </testsuites>
    
    <coverage processUncoveredFiles="true">
        <include>
            <directory suffix=".php">includes</directory>
        </include>
    </coverage>
</phpunit>
```

---

## Test Structure

```
tests/
├── Admin/
│   └── Handlers/
│       └── FormHandlerTest.php    # Admin form handling tests
├── Extensions/
│   └── ExtensionManagerTest.php   # Extension system tests
├── Layouts/
│   └── DynamicDataLayout/
│       └── LayoutRegistryTest.php   # Layout registry tests
├── Services/
│   ├── Fonts/
│   │   └── FontsRepositoryTest.php # Font management tests
│   └── GutenbergServiceTest.php   # Gutenberg service tests
├── Helpers/
│   └── TestCase.php               # Base test case class
├── bootstrap.php                  # Test bootstrap
└── phpunit.xml                    # PHPUnit configuration
```

---

## Running Tests

### Run All Tests

```bash
./vendor/bin/phpunit
```

### Run Specific Test Suite

```bash
# Admin tests only
./vendor/bin/phpunit --testsuite="Admin"

# Extension tests only
./vendor/bin/phpunit --testsuite="Extensions"

# Layout tests only
./vendor/bin/phpunit --testsuite="Layouts"
```

### Run Specific Test Class

```bash
./vendor/bin/phpunit --filter="FormHandlerTest"
./vendor/bin/phpunit --filter="ExtensionManagerTest"
./vendor/bin/phpunit --filter="LayoutRegistryTest"
```

### Run Single Test Method

```bash
./vendor/bin/phpunit --filter="testConstructor"
```

### Run with Coverage

```bash
# HTML report
./vendor/bin/phpunit --coverage-html coverage/

# Text report
./vendor/bin/phpunit --coverage-text

# XML report for CI
./vendor/bin/phpunit --coverage-xml coverage/xml/
```

---

## Writing Tests

### Basic Test Class

```php
<?php

namespace Tests\Admin\Handlers;

use Jankx\Admin\Handlers\FormHandler;
use Jankx\Foundation\Application;
use Tests\Helpers\TestCase;

class FormHandlerTest extends TestCase
{
    protected $formHandler;
    protected $app;

    protected function setUp(): void
    {
        parent::setUp();
        
        $this->app = new Application();
        $this->formHandler = new FormHandler($this->app);
    }

    public function testConstructor()
    {
        $this->assertInstanceOf(FormHandler::class, $this->formHandler);
    }

    public function testHandleRequestsProcessesValidNonce()
    {
        // Mock WordPress nonce verification
        $GLOBALS['mock_wp_verify_nonce'] = true;
        
        $_POST = [
            'jankx_action' => 'save_settings',
            'jankx_nonce' => 'valid_nonce',
        ];
        
        $result = $this->formHandler->handleRequests();
        
        $this->assertTrue($result);
    }
}
```

### Using Mockery

```php
use Mockery;

public function testExtensionActivation()
{
    // Create mock extension
    $mockExtension = Mockery::mock('Jankx\Extensions\AbstractExtension');
    $mockExtension->shouldReceive('activate')->once();
    $mockExtension->shouldReceive('getId')->andReturn('test-extension');
    
    // Register and activate
    $manager = new ExtensionManager($this->app);
    $manager->add_extension('test', $mockExtension);
    $manager->activate_extension('test');
    
    $this->assertTrue($manager->is_extension_active('test'));
}
```

### Testing with Facades

```php
use Jankx\Facades\Config;

public function testConfigFacade()
{
    // Set up facade application
    Config::setFacadeApplication($this->app);
    
    // Mock config value
    $this->app->shouldReceive('get')
        ->with('config.framework')
        ->andReturn('jankx');
    
    $this->assertEquals('jankx', Config::get('framework'));
}
```

---

## Mocking WordPress

### Available Mocks (bootstrap.php)

```php
// Actions & Filters
add_action($tag, $callback, $priority = 10, $accepted_args = 1);
add_filter($tag, $callback, $priority = 10, $accepted_args = 1);
do_action($tag, ...$args);
apply_filters($tag, $value, ...$args);

// Options
get_option($option, $default = false);
update_option($option, $value, $autoload = null);
delete_option($option);

// Nonces & Security
wp_verify_nonce($nonce, $action = -1);
wp_create_nonce($action = -1);
current_user_can($capability);

// Formatting
sanitize_key($key);
sanitize_text_field($str);
esc_html($text);
esc_html__($text, $domain = 'default');

// Time
current_time($type = 'mysql');

// URL/Path
get_template_directory();
get_stylesheet_directory();
get_template_directory_uri();
get_stylesheet_directory_uri();
```

### Custom Mock Values

```php
// Set mock values via globals
$GLOBALS['mock_wp_verify_nonce'] = true;
$GLOBALS['mock_current_user_can'] = true;

// Capture admin notices
$this->assertNotEmpty($GLOBALS['admin_notices'] ?? []);
```

### Mocking Complex Scenarios

```php
public function testFormSubmissionWithHooks()
{
    // Capture actions fired
    $actionsFired = [];
    $GLOBALS['mock_actions']['jankx/form/submitted'] = function() use (&$actionsFired) {
        $actionsFired[] = 'submitted';
    };
    
    // Submit form
    $this->formHandler->handleRequests();
    
    // Verify action was fired
    $this->assertContains('submitted', $actionsFired);
}
```

---

## Code Coverage

### Current Coverage Status

| Component | Tests | Coverage |
|-----------|-------|----------|
| Admin/Handlers | 15 | High |
| Extensions | 12 | Medium |
| Layouts | 14 | High |
| Services/Fonts | 15 | Medium |
| Services/Gutenberg | 11 | Medium |
| **Total** | **266** | **Good** |

### Generating Reports

```bash
# Generate HTML coverage report
./vendor/bin/phpunit --coverage-html coverage/

# Open in browser
open coverage/index.html
```

### Coverage Configuration

```xml
<!-- phpunit.xml -->
<coverage processUncoveredFiles="true">
    <include>
        <directory suffix=".php">includes</directory>
    </include>
    <exclude>
        <directory>includes/vendor</directory>
        <file>includes/debug.php</file>
    </exclude>
    <report>
        <html outputDirectory="coverage/"/>
        <text outputFile="php://stdout"/>
    </report>
</coverage>
```

---

## Best Practices

### 1. Test Naming

```php
// Good
public function testHandleRequestsReturnsFalseForInvalidNonce()
public function testActivateExtensionCallsActivateMethod()

// Bad
public function test1()
public function testNonce()
```

### 2. Test Structure (AAA Pattern)

```php
public function testSomething()
{
    // Arrange
    $input = 'test';
    $expected = 'TEST';
    
    // Act
    $result = strtoupper($input);
    
    // Assert
    $this->assertEquals($expected, $result);
}
```

### 3. One Concept Per Test

```php
// Good - separate concerns
public function testConstructorSetsDefaultValues()
public function testSetOptionsUpdatesValues()
public function testRenderReturnsHtmlString()

// Bad - testing multiple things
public function testEverything()
```

### 4. Use Assertions Descriptively

```php
// Good
$this->assertInstanceOf(LayoutRegistry::class, $registry);
$this->assertArrayHasKey('grid', $layouts);
$this->assertStringContainsString('Image size settings saved', $output);

// Use custom messages when needed
$this->assertNotEmpty(
    $GLOBALS['admin_notices'] ?? [], 
    'admin_notices array should not be empty'
);
```

### 5. Clean Up Resources

```php
protected function tearDown(): void
{
    // Clean up superglobals
    $_POST = [];
    $_GET = [];
    $_SERVER['REQUEST_METHOD'] = 'GET';
    
    // Reset mock globals
    $GLOBALS['mock_wp_verify_nonce'] = false;
    $GLOBALS['mock_current_user_can'] = false;
    
    // Close Mockery
    Mockery::close();
    
    parent::tearDown();
}
```

### 6. Use Data Providers

```php
/**
 * @dataProvider layoutProvider
 */
public function testLayoutCanBeResolved(string $layoutName, string $layoutClass)
{
    $layout = $this->registry->resolve($layoutName);
    $this->assertInstanceOf($layoutClass, $layout);
}

public function layoutProvider(): array
{
    return [
        ['grid', GridLayout::class],
        ['list', ListLayout::class],
        ['carousel', CarouselLayout::class],
    ];
}
```

---

## Troubleshooting

### Common Issues

**Issue**: `A facade root has not been set`
```php
// Solution: Set facade application in setUp()
\Jankx\Facades\App::setFacadeApplication($this->app);
```

**Issue**: `Call to undefined function`
```php
// Solution: Add mock to bootstrap.php
if (!function_exists('my_wp_function')) {
    function my_wp_function() {
        return $GLOBALS['mock_my_wp_function'] ?? null;
    }
}
```

**Issue**: `InvalidCountException` with Mockery
```php
// Solution: Close Mockery properly
tearDown() {
    Mockery::close();
    parent::tearDown();
}
```

---

## Continuous Integration

### GitHub Actions Example

```yaml
# .github/workflows/tests.yml
name: Tests

on: [push, pull_request]

jobs:
  test:
    runs-on: ubuntu-latest
    
    steps:
    - uses: actions/checkout@v2
    
    - name: Setup PHP
      uses: shivammathur/setup-php@v2
      with:
        php-version: '7.4'
        extensions: mbstring, intl
        coverage: xdebug
    
    - name: Install dependencies
      run: composer install
    
    - name: Run tests
      run: ./vendor/bin/phpunit --coverage-text
```

---

## Resources

- [PHPUnit Documentation](https://phpunit.readthedocs.io/)
- [Mockery Documentation](https://docs.mockery.io/)
- [WordPress PHPUnit](https://make.wordpress.org/core/handbook/testing/automated-testing/)
- [Testing Best Practices](../architecture/testing-patterns.md)
