# Hướng Dẫn Kiểm Thử

Tài liệu kiểm thử toàn diện cho Jankx Theme.

## Tổng Quan

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

## Mục Lục

- [Bắt Đầu](#bắt-đầu)
- [Cấu Trúc Test](#cấu-trúc-test)
- [Chạy Tests](#chạy-tests)
- [Viết Tests](#viết-tests)
- [Code Coverage](#code-coverage)

---

## Bắt Đầu

### Cài Đặt

```bash
# Cài PHPUnit và dependencies
composer install --dev

# Kiểm tra cài đặt
./vendor/bin/phpunit --version
```

---

## Cấu Trúc Test

```
tests/
├── Admin/
│   └── Handlers/
│       └── FormHandlerTest.php
├── Extensions/
│   └── ExtensionManagerTest.php
├── Layouts/
│   └── DynamicDataLayout/
│       └── LayoutRegistryTest.php
├── Services/
│   ├── Fonts/
│   │   └── FontsRepositoryTest.php
│   └── GutenbergServiceTest.php
├── Helpers/
│   └── TestCase.php
├── bootstrap.php
└── phpunit.xml
```

---

## Chạy Tests

### Chạy Tất Cả

```bash
./vendor/bin/phpunit
```

### Chạy Test Suite Cụ Thể

```bash
# Admin tests
./vendor/bin/phpunit --testsuite="Admin"

# Extension tests
./vendor/bin/phpunit --testsuite="Extensions"

# Layout tests
./vendor/bin/phpunit --testsuite="Layouts"
```

### Chạy Test Class

```bash
./vendor/bin/phpunit --filter="FormHandlerTest"
./vendor/bin/phpunit --filter="ExtensionManagerTest"
```

### Chạy với Coverage

```bash
# HTML report
./vendor/bin/phpunit --coverage-html coverage/

# Text report
./vendor/bin/phpunit --coverage-text
```

---

## Viết Tests

### Class Test Cơ Bản

```php
<?php
namespace Tests\Admin\Handlers;

use Jankx\Admin\Handlers\FormHandler;
use Tests\Helpers\TestCase;

class FormHandlerTest extends TestCase
{
    protected $formHandler;

    protected function setUp(): void
    {
        parent::setUp();
        $this->formHandler = new FormHandler($this->app);
    }

    public function testConstructor()
    {
        $this->assertInstanceOf(FormHandler::class, $this->formHandler);
    }
}
```

### Sử Dụng Mockery

```php
use Mockery;

public function testExtensionActivation()
{
    $mockExtension = Mockery::mock('Jankx\Extensions\AbstractExtension');
    $mockExtension->shouldReceive('activate')->once();
    
    $manager = new ExtensionManager($this->app);
    $manager->add_extension('test', $mockExtension);
    $manager->activate_extension('test');
    
    $this->assertTrue($manager->is_extension_active('test'));
}
```

---

## Code Coverage

### Trạng Thái Hiện Tại

| Component | Tests | Coverage |
|-----------|-------|----------|
| Admin/Handlers | 15 | Cao |
| Extensions | 12 | Trung bình |
| Layouts | 14 | Cao |
| Services/Fonts | 15 | Trung bình |
| Services/Gutenberg | 11 | Trung bình |
| **Tổng** | **266** | **Tốt** |

### Tạo Báo Cáo

```bash
# Tạo HTML coverage report
./vendor/bin/phpunit --coverage-html coverage/

# Mở trong browser
open coverage/index.html
```

---

## Best Practices

### 1. Đặt Tên Test

```php
// Tốt
public function testHandleRequestsReturnsFalseForInvalidNonce()
public function testActivateExtensionCallsActivateMethod()

// Xấu
public function test1()
public function testNonce()
```

### 2. Cấu Trúc Test (AAA Pattern)

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

### 3. Dọn Dẹp Resources

```php
protected function tearDown(): void
{
    $_POST = [];
    $_GET = [];
    $GLOBALS['mock_wp_verify_nonce'] = false;
    Mockery::close();
    parent::tearDown();
}
```

---

## Continuous Integration

### GitHub Actions

```yaml
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

## Tài Nguyên

- [PHPUnit Documentation](https://phpunit.readthedocs.io/)
- [Mockery Documentation](https://docs.mockery.io/)
