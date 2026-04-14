# Đóng Góp cho Jankx

Cảm ơn bạn đã quan tâm đến việc đóng góp cho Jankx! Hướng dẫn này sẽ giúp bạn bắt đầu.

## Quy Tắc Ứng Xử

- Tôn trọng và hòa nhập
- Đưa ra phản hồi mang tính xây dựng
- Tập trung vào lợi ích của cộng đồng

## Tôi Có Thể Đóng Góp Như Thế Nào?

### Báo Cáo Lỗi

Trước khi tạo báo cáo lỗi:

1. Kiểm tra xem vấn đề đã tồn tại chưa
2. Xác minh bằng phiên bản mới nhất
3. Thu thập thông tin liên quan

**Mẫu báo cáo lỗi:**
```
**Mô tả:**
Mô tả rõ ràng về lỗi

**Các Bước Tái Hiện:**
1. Vào '...'
2. Click vào '...'
3. Thấy lỗi

**Hành Vi Mong Đợi:**
Điều gì nên xảy ra

**Môi Trường:**
- WordPress version:
- PHP version:
- Trình duyệt:
```

### Đề Xuất Tính Năng

- Giải thích tại sao tính năng này hữu ích
- Cung cấp ví dụ về cách hoạt động
- Xem xét phạm vi và độ phức tạp

### Pull Requests

1. Fork repository
2. Tạo branch (`git checkout -b feature/tinh-nang-hay`)
3. Thay đổi code
4. Chạy tests (`./vendor/bin/phpunit`)
5. Commit (`git commit -m 'Thêm tính năng hay'`)
6. Push (`git push origin feature/tinh-nang-hay`)
7. Mở Pull Request

## Thiết Lập Môi Trường Phát Triển

```bash
# Clone fork của bạn
git clone https://github.com/YOUR_USERNAME/theme.git

# Cài dependencies
composer install
npm install

# Thiết lập WordPress testing (cần WP local)
cd /path/to/wordpress/wp-content/themes/jankx

# Chạy tests
./vendor/bin/phpunit

# Chạy tests với coverage
./vendor/bin/phpunit --coverage-html coverage/
```

## Tiêu Chuẩn Code

### PHP

- Tuân theo PSR-12
- Dùng strict typing khi có thể: `declare(strict_types=1);`
- Viết docblock cho tất cả methods
- Độ dài dòng tối đa: 120 ký tự

```php
<?php
declare(strict_types=1);

namespace Jankx\Features;

/**
 * Class ví dụ về tiêu chuẩn code
 */
class ExampleFeature
{
    /**
     * Xử lý dữ liệu và trả về kết quả định dạng
     *
     * @param array $data Dữ liệu đầu vào
     * @param string $format Định dạng đầu ra
     * @return string Kết quả đã định dạng
     */
    public function process(array $data, string $format): string
    {
        // Implementation
    }
}
```

### JavaScript

- Dùng ES6+ features
- Theo tiêu chuẩn WordPress JavaScript
- Chạy ESLint trước khi commit

```bash
npm run lint:js
npm run lint:css
```

### CSS/SCSS

- Dùng BEM naming convention
- Mobile-first approach
- CSS variables cho theming

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

### Viết Tests

Tất cả tính năng mới phải có tests:

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

- Mục tiêu 80%+ coverage cho code mới
- Critical paths phải có 100% coverage
- Dùng Mockery cho mocking

## Tài Liệu

- Cập nhật tài liệu liên quan cho tính năng mới
- Thêm PHPDoc blocks cho tất cả public methods
- Bao gồm code examples khi hữu ích

## Quy Tắc Commit Message

Dùng định dạng conventional commits:

```
type(scope): subject

body (tùy chọn)

footer (tùy chọn)
```

**Các loại:**
- `feat`: Tính năng mới
- `fix`: Sửa lỗi
- `docs`: Chỉ tài liệu
- `style`: Code style (formatting, không đổi logic)
- `refactor`: Tái cấu trúc code
- `test`: Thêm hoặc cập nhật tests
- `chore`: Công việc bảo trì

**Ví dụ:**
```
feat(layout): thêm hỗ trợ masonry layout

fix(admin): sửa lỗi hiển thị notice

docs: cập nhật hướng dẫn đóng góp
```

## Phát Triển Extension

Muốn tạo extension? Xem [Hướng Dẫn Extensions](../extensions/).

Bắt đầu nhanh:
```php
namespace Jankx\Extensions\MyExtension;

use Jankx\Extensions\AbstractExtension;

class MyExtension extends AbstractExtension
{
    protected $id = 'my-extension';
    protected $name = 'My Extension';
    
    public function activate(): void
    {
        // Code của bạn
    }
}
```

## Câu Hỏi?

- GitHub Discussions: https://github.com/jankx/jankx-lite/discussions
- Issues: https://github.com/jankx/jankx-lite/issues

## License

Bằng cách đóng góp, bạn đồng ý rằng các đóng góp của bạn sẽ được cấp phép theo GPL-2.0+ License.
