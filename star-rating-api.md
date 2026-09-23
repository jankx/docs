# Star Rating API Documentation

Hệ thống Rating REST API cho Jankx Theme — cung cấp khả năng đánh giá sao (star rating) trên tất cả post types.

---

## Table of Contents

1. [Overview](#overview)
2. [Architecture](#architecture)
3. [REST API](#rest-api)
   - [POST /wp-json/jankx/v1/star-rating/submit](#post-submit)
   - [GET /wp-json/jankx/v1/star-rating/providers](#get-providers)
4. [Shortcode](#shortcode)
5. [Gutenberg Block](#gutenberg-block)
6. [Extending the System](#extending)
   - [Registering a Custom Provider](#registering-custom-provider)
   - [ConfigurableRatingProvider](#configurable-provider)
   - [Implementing StarRatingProviderInterface](#implementing-interface)
7. [Frontend React Component](#react-component)
8. [Hooks & Filters](#hooks)
9. [Data Model](#data-model)
10. [Post Types & Meta Keys](#post-types)

---

## Overview

Star Rating API cho phép:

- **Đọc rating** từ database qua Providers (Strategy Pattern)
- **Ghi rating** qua REST API (tạo comment type `review`)
- **Hiển thị form** qua Shortcode hoặc React Component
- **Mở rộng** bằng cách register Provider mới

### Tech Stack

- **Backend:** PHP 8.x, WordPress REST API
- **Frontend:** React 18, TypeScript
- **Pattern:** Strategy + Registry (Singleton)

---

## Architecture

```
┌─────────────────────────────────────────────────────────┐
│                    Gutenberg Block                       │
│                  (jankx/star-rating)                     │
└─────────────────────┬───────────────────────────────────┘
                      │
                      ▼
┌─────────────────────────────────────────────────────────┐
│                   StarRatingRegistry                     │
│              (Singleton, manages providers)               │
└──────┬────────────┬────────────┬────────────┬───────────┘
       │            │            │            │
       ▼            ▼            ▼            ▼
┌──────────┐ ┌──────────┐ ┌──────────┐ ┌──────────────────┐
│  Manual   │ │WooCommerce│ │ PostMeta │ │  Configurable    │
│  Provider │ │ Provider  │ │ Provider │ │  Provider        │
└──────────┘ └──────────┘ └──────────┘ └──────────────────┘
                                                      │
                    ┌─────────────────────────────────┘
                    ▼
           ┌────────────────┐
           │   Extensions   │
           │ tour / place / │
           │ product / ...  │
           └────────────────┘

┌─────────────────────────────────────────────────────────┐
│              RatingSubmission (REST API)                 │
│         POST /wp-json/jankx/v1/star-rating/submit        │
└─────────────────────┬───────────────────────────────────┘
                      │
                      ▼
┌─────────────────────────────────────────────────────────┐
│            WordPress Comment System                      │
│         (comment type: review)                           │
└─────────────────────┬───────────────────────────────────┘
                      │
                      ▼
┌─────────────────────────────────────────────────────────┐
│            RatingRepository (comment-rating ext)         │
│      Computes: jankx_rating_average, jankx_rating_count │
│      Syncs: _tour_rating, _place_rating, ...            │
└─────────────────────────────────────────────────────────┘
```

---

## REST API

<a id="post-submit"></a>
### POST /wp-json/jankx/v1/star-rating/submit

Gửi đánh giá sao cho một bài viết.

#### Endpoint

```
POST /wp-json/jankx/v1/star-rating/submit
```

#### Request Body

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `post_id` | integer | **Yes** | ID của bài viết cần đánh giá |
| `rating` | number | **Yes** | Điểm đánh giá (1-5, hoặc 1-10 nếu config) |
| `review` | string | No | Nội dung đánh giá (text) |
| `pros` | string | No | Điểm mạnh, mỗi dòng một ý |
| `cons` | string | No | Điểm yếu, mỗi dòng một ý |
| `author_name` | string | No | Tên tác giả (cho guest) |
| `author_email` | string | No | Email tác giả (cho guest) |

#### Request Example

**Logged-in user:**

```bash
curl -X POST https://nibitour.vn/wp-json/jankx/v1/star-rating/submit \
  -H "Content-Type: application/json" \
  -H "X-WP-Nonce: YOUR_NONCE" \
  -d '{
    "post_id": 123,
    "rating": 4,
    "review": "Tour rất tốt, hướng dẫn viên nhiệt tình!",
    "pros": "Hướng dẫn viên nhiệt tình\nPhòng đẹp\nĐồ ăn ngon",
    "cons": "Lịch trình hơi gấp"
  }'
```

**Guest (không đăng nhập):**

```bash
curl -X POST https://nibitour.vn/wp-json/jankx/v1/star-rating/submit \
  -H "Content-Type: application/json" \
  -d '{
    "post_id": 123,
    "rating": 5,
    "review": "Excellent!",
    "author_name": "Nguyễn Văn A",
    "author_email": "example@email.com"
  }'
```

#### Response

**Success (200):**

```json
{
  "success": true,
  "message": "Đánh giá đã được gửi thành công!",
  "rating": {
    "average": 4.2,
    "count": 15,
    "user_rating": 4,
    "comment_id": 456
  }
}
```

**Already Rated (409):**

```json
{
  "success": false,
  "message": "Bạn đã đánh giá bài viết này rồi.",
  "existing": {
    "rating": 5,
    "comment_id": 789
  }
}
```

**Post Not Found (404):**

```json
{
  "success": false,
  "message": "Bài viết không tồn tại."
}
```

**Server Error (500):**

```json
{
  "success": false,
  "message": "Không thể lưu đánh giá. Vui lòng thử lại."
}
```

#### Permission

- Endpoint mở cho tất cả (`permission_callback: __return_true`)
- Guest cần cung cấp `author_name` và `author_email`
- User đăng nhập sẽ tự động lấy tên/email từ tài khoản

---

<a id="get-providers"></a>
### GET /wp-json/jankx/v1/star-rating/providers

Lấy danh sách các rating providers có sẵn.

#### Endpoint

```
GET /wp-json/jankx/v1/star-rating/providers?post_type=tour
```

#### Query Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `post_type` | string | No | Lọc providers theo post type |

#### Response

```json
[
  {
    "value": "tour_rating",
    "label": "Tour Rating",
    "editorConfig": [
      {
        "type": "toggle",
        "attribute": "showCount",
        "label": "Show review count"
      }
    ]
  },
  {
    "value": "manual",
    "label": "Manual",
    "editorConfig": [
      {
        "type": "range",
        "attribute": "manualRating",
        "label": "Rating Value",
        "min": 0,
        "max": 5,
        "step": 0.1
      }
    ]
  },
  {
    "value": "woocommerce",
    "label": "WooCommerce Product Rating",
    "editorConfig": []
  }
]
```

---

## Shortcode

### `[jankx_rating_form]`

Embed form đánh giá sao trên bất kỳ trang nào.

#### Parameters

| Parameter | Default | Description |
|-----------|---------|-------------|
| `post_id` | Current post | ID bài viết (tự detect nếu ở trang singular) |
| `max_rating` | `5` | Số sao tối đa (1-10) |
| `show_review` | `yes` | Hiển thị ô nhập đánh giá |
| `show_pros_cons` | `no` | Hiển thị ô nhập pros/cons |

#### Examples

```php
// Form cơ bản
[jankx_rating_form]

// Form cho bài viết cụ thể
[jankx_rating_form post_id="123"]

// Form với 10 sao và pros/cons
[jankx_rating_form max_rating="10" show_pros_cons="yes"]

// Chỉ hiển thị rating, không có review text
[jankx_rating_form show_review="no"]
```

---

## Gutenberg Block

### Block: `jankx/star-rating`

Hiển thị rating sao trên trang.

#### Attributes

| Attribute | Type | Default | Description |
|-----------|------|---------|-------------|
| `ratingSource` | string | `'manual'` | Nguồn dữ liệu rating |
| `manualRating` | number | `5` | Giá trị rating thủ công |
| `displayStyle` | string | `'stars'` | Kiểu hiển thị: `stars` hoặc `summary` |
| `starSize` | number | `16` | Kích thước sao (px) |
| `starColor` | string | `'#f1c40f'` | Màu sao |
| `starEmptyColor` | string | `'#dddddd'` | Màu sao trống |
| `showCount` | boolean | `false` | Hiển thị số lượng đánh giá |
| `align` | string | `'left'` | Căn lề: `left`, `center`, `right` |
| `iconType` | string | `'text'` | Kiểu icon: `text` hoặc `svg` |

#### Rating Sources

Provider được đăng ký tự động bởi các extensions:

| Source ID | Label | Post Types | Meta Keys |
|-----------|-------|------------|-----------|
| `tour_rating` | Tour Rating | `tour` | `jankx_rating_average`, `jankx_rating_count` |
| `place_rating` | Place Rating | `place` | `jankx_rating_average`, `jankx_rating_count` |
| `service_rating` | Service Rating | `service` | `jankx_rating_average`, `jankx_rating_count` |
| `product_rating` | Product Rating | `product` | `jankx_rating_average`, `jankx_rating_count` |
| `experience_rating` | Experience Rating | `experience` | `jankx_rating_average`, `jankx_rating_count` |
| `manual` | Manual | all | `manualRating` attribute |
| `woocommerce` | WooCommerce | `product` | WooCommerce rating |
| `post_meta` | Post Meta | all | configurable meta keys |

---

## Extending the System

<a id="registering-custom-provider"></a>
### Registering a Custom Provider

Cách nhanh nhất để thêm rating source mới cho extension:

```php
// Trong extension's register_hooks()
add_action('init', function () {
    \Jankx\Gutenberg\StarRating\StarRatingRegistry::register(
        new \Jankx\Gutenberg\StarRating\Providers\ConfigurableRatingProvider([
            'id'             => 'my_extension_rating',
            'label'          => __('My Extension Rating', 'my-textdomain'),
            'postTypes'      => ['my_post_type'],
            'ratingMetaKey'  => 'jankx_rating_average',
            'countMetaKey'   => 'jankx_rating_count',
            'editorControls' => [
                [
                    'type'      => 'toggle',
                    'attribute' => 'showCount',
                    'label'     => __('Show count', 'my-textdomain'),
                ],
            ],
        ])
    );
}, 20);
```

<a id="configurable-provider"></a>
### ConfigurableRatingProvider

Config array reference:

```php
[
    // Required
    'id'             => 'string',      // Unique provider ID
    'label'          => 'string',      // Human-readable label for editor

    // Post type filtering
    'postTypes'      => ['string'],    // Supported post types ([] = all)

    // Meta keys
    'ratingMetaKey'  => 'string',      // Post meta key for average rating
    'countMetaKey'   => 'string',      // Post meta key for review count

    // Editor controls (declarative UI)
    'editorControls' => [
        [
            'type'      => 'range',    // 'range', 'text', 'select', 'toggle'
            'attribute' => 'string',   // Block attribute name
            'label'     => 'string',   // Control label
            'help'      => 'string',   // Help text (optional)
            'default'   => mixed,      // Default value (optional)
            'min'       => number,     // Range: min value
            'max'       => number,     // Range: max value
            'step'      => number,     // Range: step value
            'options'   => [           // Select: options array
                ['value' => 'string', 'label' => 'string'],
            ],
        ],
    ],
]
```

<a id="implementing-interface"></a>
### Implementing StarRatingProviderInterface

Để tạo custom provider với logic phức tạp hơn:

```php
<?php

namespace Jankx\Extensions\MyExtension\Rating;

use Jankx\Gutenberg\StarRating\StarRatingProviderInterface;

class CustomRatingProvider implements StarRatingProviderInterface
{
    public function getId(): string
    {
        return 'my_custom_rating';
    }

    public function getLabel(): string
    {
        return __('My Custom Rating', 'my-textdomain');
    }

    public function getSupportedPostTypes(): array
    {
        return ['my_post_type'];
    }

    public function getRating(int $postId, array $attributes): float
    {
        // Custom logic to get rating
        // Ví dụ: query từ external API,计算 từ multiple sources, etc.
        $rating = get_post_meta($postId, '_my_custom_rating', true);
        return max(0.0, min(5.0, (float) $rating));
    }

    public function getCount(int $postId, array $attributes): int
    {
        return (int) get_post_meta($postId, '_my_custom_review_count', true);
    }

    public function getEditorConfig(): array
    {
        return [
            [
                'type'      => 'select',
                'attribute' => 'ratingSource',
                'label'     => __('Source Type', 'my-textdomain'),
                'options'   => [
                    ['value' => 'meta', 'label' => __('Post Meta', 'my-textdomain')],
                    ['value' => 'api', 'label' => __('External API', 'my-textdomain')],
                ],
            ],
            [
                'type'      => 'text',
                'attribute' => 'apiUrl',
                'label'     => __('API URL', 'my-textdomain'),
                'default'   => 'https://api.example.com/ratings',
            ],
        ];
    }
}
```

Register:

```php
add_action('init', function () {
    \Jankx\Gutenberg\StarRating\StarRatingRegistry::register(
        new CustomRatingProvider()
    );
}, 20);
```

---

<a id="react-component"></a>
### Frontend React Component

`RatingSubmissionForm` component để embed form rating trên frontend:

```tsx
import { RatingSubmissionForm } from './components/RatingSubmissionForm';

// Usage
<RatingSubmissionForm
    postId={123}
    maxRating={5}
    showReview={true}
    showProsCons={false}
    onSuccess={(data) => {
        console.log('Rating submitted!', data.rating.average);
    }}
    onError={(error) => {
        console.error('Error:', error);
    }}
    translations={{
        submit: 'Gửi đánh giá',
        success: 'Cảm ơn bạn!',
    }}
/>
```

#### Props

| Prop | Type | Default | Description |
|------|------|---------|-------------|
| `postId` | number | **required** | ID bài viết |
| `maxRating` | number | `5` | Số sao tối đa |
| `showReview` | boolean | `true` | Hiển thị ô review |
| `showProsCons` | boolean | `false` | Hiển thị ô pros/cons |
| `restUrl` | string | auto | REST API URL |
| `nonce` | string | auto | WP REST nonce |
| `onSuccess` | function | - | Callback khi thành công |
| `onError` | function | - | Callback khi lỗi |
| `translations` | object | - | Custom translations |

---

<a id="hooks"></a>
## Hooks & Filters

### Actions

| Hook | Parameters | Description |
|------|------------|-------------|
| `jankx/star_rating/submitted` | `$commentId`, `$postId`, `$rating`, `$request` | Fires sau khi rating được submit thành công |
| `jankx/review_system/rating_synced` | `$commentId`, `$postId`, `$rating` | Fires khi review-system sync rating |

### Filters

| Hook | Parameters | Description |
|------|------------|-------------|
| `jankx/star_rating/value` | `$rating`, `$attributes`, `$postId` | Override rating value trước khi render |
| `jankx/star_rating/count` | `$count`, `$attributes`, `$postId` | Override count trước khi render |
| `jankx/star_rating/unknown_provider_rating` | `$rating`, `$source`, `$postId`, `$attributes` | Handle unknown provider ID |
| `jankx/comment_rating/legacy_meta_map` | `$map` | Override legacy meta key mapping |

### Example: Hook into rating submission

```php
add_action('jankx/star_rating/submitted', function ($commentId, $postId, $rating, $request) {
    // Gửi email thông báo
    // Cập nhật analytics
    // Trigger webhook
    // etc.
}, 10, 4);
```

---

<a id="data-model"></a>
## Data Model

### Comment Meta

| Key | Type | Description |
|-----|------|-------------|
| `jankx_comment_rating` | int | Điểm rating (1-5 hoặc 1-10) |
| `jankx_comment_post_id` | int | Post ID mà comment thuộc về |
| `_jankx_review_pros` | array | Danh sách điểm mạnh |
| `_jankx_review_cons` | array | Danh sách điểm yếu |

### Post Meta (Aggregates)

| Key | Type | Description |
|-----|------|-------------|
| `jankx_rating_average` | float | Điểm trung bình (canonical) |
| `jankx_rating_count` | int | Tổng số đánh giá (canonical) |
| `jankx_rating_values` | array | Map `{comment_id: rating}` |

### Post Meta (Legacy Display)

| Key | Post Type | Description |
|-----|-----------|-------------|
| `_tour_rating` | `tour` | Rating display cho tour |
| `_experience_rating` | `experience` | Rating display cho experience |
| `_place_rating` | `place` | Rating display cho place |
| `_tour_review_count` | `tour` | Số review (admin editable) |

### Data Flow

```
User submits rating
        │
        ▼
REST API creates comment (type: review)
        │
        ▼
RatingRepository::save()
        │
        ├─► update_comment_meta(comment_id, 'jankx_comment_rating', rating)
        │
        └─► recomputeAggregate(post_id)
                │
                ├─► update_post_meta(post_id, 'jankx_rating_average', avg)
                ├─► update_post_meta(post_id, 'jankx_rating_count', count)
                ├─► update_post_meta(post_id, 'jankx_rating_values', values)
                │
                └─► syncLegacyRatingMeta()
                        │
                        ├─► _tour_rating (for tour posts)
                        ├─► _experience_rating (for experience posts)
                        └─► _place_rating (for place posts)
```

---

<a id="post-types"></a>
## Post Types & Meta Keys

### Extension: Travel (Tour)

| Meta Key | Type | Description |
|----------|------|-------------|
| `jankx_rating_average` | float | Rating trung bình (từ comment-rating) |
| `jankx_rating_count` | int | Số lượng review (từ comment-rating) |
| `_tour_rating` | float | Legacy display rating |
| `_tour_review_count` | int | Số review (admin editable) |

### Extension: Place

| Meta Key | Type | Description |
|----------|------|-------------|
| `jankx_rating_average` | float | Rating trung bình |
| `jankx_rating_count` | int | Số lượng review |
| `_place_rating` | float | Legacy display rating |

### Extension: Service

| Meta Key | Type | Description |
|----------|------|-------------|
| `jankx_rating_average` | float | Rating trung bình |
| `jankx_rating_count` | int | Số lượng review |

### Extension: Product

| Meta Key | Type | Description |
|----------|------|-------------|
| `jankx_rating_average` | float | Rating trung bình |
| `jankx_rating_count` | int | Số lượng review |
| `_product_rating` | float | Legacy display rating |

### Extension: Experience

| Meta Key | Type | Description |
|----------|------|-------------|
| `jankx_rating_average` | float | Rating trung bình |
| `jankx_rating_count` | int | Số lượng review |
| `_experience_rating` | float | Legacy display rating |

---

## PHP Files Reference

| File | Description |
|------|-------------|
| `StarRatingProviderInterface.php` | Interface cho providers |
| `AbstractStarRatingProvider.php` | Abstract base class |
| `StarRatingRegistry.php` | Singleton registry |
| `RatingSubmission.php` | REST API endpoint |
| `RatingSubmissionShortcode.php` | `[jankx_rating_form]` shortcode |
| `Providers/ConfigurableRatingProvider.php` | Data-driven provider |
| `Providers/ManualRatingProvider.php` | Manual rating |
| `Providers/WooCommerceRatingProvider.php` | WooCommerce integration |
| `Providers/PostMetaRatingProvider.php` | Custom meta key |
| `Providers/CrawlerRatingProvider.php` | Crawler/scraper rating |

---

## Example: Complete Extension Integration

```php
<?php
// File: extensions/my-tours/MyToursExtension.php

namespace Jankx\Extensions\MyTours;

use Jankx\Extensions\AbstractExtension;

class MyToursExtension extends AbstractExtension
{
    public function register_hooks(): void
    {
        // Register post type, taxonomy, etc.
        (new TourPostType())->register();

        // Register star rating provider
        add_action('init', [$this, 'register_star_rating_provider'], 20);

        // Hook into rating submissions
        add_action('jankx/star_rating/submitted', [$this, 'on_rating_submitted'], 10, 4);
    }

    public function register_star_rating_provider(): void
    {
        if (!class_exists('\Jankx\Gutenberg\StarRating\StarRatingRegistry')) {
            return;
        }

        \Jankx\Gutenberg\StarRating\StarRatingRegistry::register(
            new \Jankx\Gutenberg\StarRating\Providers\ConfigurableRatingProvider([
                'id'             => 'my_tour_rating',
                'label'          => __('My Tour Rating', 'my-textdomain'),
                'postTypes'      => ['my_tour'],
                'ratingMetaKey'  => 'jankx_rating_average',
                'countMetaKey'   => 'jankx_rating_count',
                'editorControls' => [
                    [
                        'type'      => 'toggle',
                        'attribute' => 'showCount',
                        'label'     => __('Show review count', 'my-textdomain'),
                    ],
                ],
            ])
        );
    }

    public function on_rating_submitted(int $commentId, int $postId, int $rating, $request): void
    {
        // Custom logic after rating submission
        // Ví dụ: gửi email, update analytics, trigger webhook
    }
}
```
