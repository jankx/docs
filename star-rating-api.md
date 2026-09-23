# Star Rating & Review System Documentation

Hệ thống Rating & Review cho Jankx Theme — cung cấp khả năng đánh giá sao (star rating) và form review trên tất cả post types.

---

## Table of Contents

1. [Overview](#overview)
2. [Architecture](#architecture)
3. [Blocks](#blocks)
   - [`jankx/star-rating`](#block-star-rating) — Hiển thị rating sao
   - [`jankx/review-form`](#block-review-form) — Form đánh giá cho user
4. [REST API](#rest-api)
   - [POST /wp-json/jankx/v1/star-rating/submit](#post-submit)
   - [GET /wp-json/jankx/v1/star-rating/providers](#get-providers)
5. [Shortcode](#shortcode)
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

Star Rating & Review System cho phép:

- **Đọc rating** từ database qua Providers (Strategy Pattern)
- **Ghi rating** qua REST API (tạo comment type `review`)
- **Hiển thị星星** qua `jankx/star-rating` block
- **Nhận review** qua `jankx/review-form` block
- **Mở rộng** bằng cách register Provider mới

### Tech Stack

- **Backend:** PHP 8.x, WordPress REST API
- **Frontend:** React 18, TypeScript
- **Pattern:** Strategy + Registry (Singleton)

---

## Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                       Gutenberg Blocks                           │
│  ┌──────────────────┐          ┌──────────────────┐            │
│  │  jankx/star-rating│          │ jankx/review-form │            │
│  │  (Display rating) │          │ (Submit rating)   │            │
│  └────────┬─────────┘          └────────┬─────────┘            │
│           │                              │                      │
│           │  GET providers               │  POST submit         │
│           │                              │                      │
└───────────┼──────────────────────────────┼──────────────────────┘
            │                              │
            ▼                              ▼
┌─────────────────────────────────────────────────────────────────┐
│                    StarRatingRegistry                            │
│               (Singleton, manages providers)                      │
└──────┬────────────┬────────────┬────────────┬────────────────────┘
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

┌─────────────────────────────────────────────────────────────────┐
│              RatingSubmission (REST API)                         │
│         POST /wp-json/jankx/v1/star-rating/submit                │
└─────────────────────┬───────────────────────────────────────────┘
                      │
                      ▼
┌─────────────────────────────────────────────────────────────────┐
│            WordPress Comment System                              │
│         (comment type: review)                                   │
└─────────────────────┬───────────────────────────────────────────┘
                      │
                      ▼
┌─────────────────────────────────────────────────────────────────┐
│            RatingRepository (comment-rating ext)                 │
│      Computes: jankx_rating_average, jankx_rating_count         │
│      Syncs: _tour_rating, _place_rating, ...                    │
└─────────────────────────────────────────────────────────────────┘
```

---

## Blocks

<a id="block-star-rating"></a>
### Block: `jankx/star-rating`

Hiển thị rating sao (stars hoặc summary) trên trang.

#### Khi nào dùng

- Hiển thị rating hiện tại của bài viết
- Component read-only, không có form nhập

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

#### Style Presets

Block cung cấp 5 preset có sẵn:

| Preset | Style | Mô tả |
|--------|-------|-------|
| `stars-default` | ★★★★☆ | Stars cơ bản |
| `stars-with-count` | ★★★★☆ (123) | Stars + số review |
| `summary-default` | ★ 4.5 (123) | Summary format |
| `google-summary` | ★ 4.6 (39,092) | Google-style summary |
| `compact-stars` | ★★★★★ | Nhỏ gọn |

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

#### Files

| File | Path | Description |
|------|------|-------------|
| `block.json` | `resources/blocks/star-rating/block.json` | Block metadata |
| `edit.tsx` | `resources/blocks/star-rating/edit.tsx` | Editor component |
| `index.tsx` | `resources/blocks/star-rating/index.tsx` | Block registration |
| `style.scss` | `resources/blocks/star-rating/style.scss` | Frontend styles |
| `editor.scss` | `resources/blocks/star-rating/editor.scss` | Editor styles |
| `StarRatingBlock.php` | `jankx/.../Blocks/StarRatingBlock.php` | Server-side render |

---

<a id="block-review-form"></a>
### Block: `jankx/review-form`

Form đánh giá sao cho user submit review.

#### Khi nào dùng

- Trang single post (tour, experience, place, product, service)
- Section "Viết đánh giá" ở cuối bài
- Kết hợp với `jankx/star-rating` để hiển thị rating hiện tại

#### Attributes

| Attribute | Type | Default | Description |
|-----------|------|---------|-------------|
| `postId` | number | `0` | Post ID (auto-detect từ editor) |
| `maxRating` | number | `5` | Số sao tối đa (3-10) |
| `showReview` | boolean | `true` | Hiển thị textarea review |
| `showProsCons` | boolean | `false` | Hiển thị ô Pros/Cons |
| `showLoginForm` | boolean | `true` | Hiển thị login cho guest |
| `formTitle` | string | `''` | Tiêu đề form (default: "Đánh giá của bạn") |
| `submitText` | string | `''` | Text nút submit (default: "Gửi đánh giá") |
| `starSize` | number | `32` | Kích thước sao (px) |
| `starColor` | string | `'#f1c40f'` | Màu sao |
| `starEmptyColor` | string | `'#dddddd'` | Màu sao trống |
| `customCSS` | string | `''` | Custom CSS cho form |

#### Editor Features

- **Preview** — Stars interactive với hover/click trong editor
- **Inspector Controls** — Panel bên phải để config
- **Post selector** — Auto-detect post ID từ editor
- **Live preview** — Thay đổi settings thấy ngay

#### Frontend Features

- **Stars interactive** — Hover + click để chọn rating
- **Review textarea** — Nhận xét tự do
- **Pros/Cons** — 2 cột nhập điểm mạnh/yếu
- **Guest support** — Nhập tên, email cho khách
- **Login link** — Liên kết đăng nhập
- **AJAX submit** — Gửi không reload trang
- **Success/error messages** — Thông báo kết quả
- **Auto-update** — Cập nhật `jankx/star-rating` block nếu có trên cùng post

#### Workflow

```
User mở trang single post
        │
        ▼
Nhìn thấy form "Đánh giá của bạn"
        │
        ├─ Đã đăng nhập → Hiển thị tên + avatar
        │
        └─ Chưa đăng nhập → Hiển thị link "Đăng nhập" + guest fields
                │
                ▼
        Chọn số sao (click/hover)
                │
                ▼
        Nhập review text (optional)
                │
                ▼
        Nhập Pros/Cons (nếu bật)
                │
                ▼
        Click "Gửi đánh giá"
                │
                ▼
        POST /wp-json/jankx/v1/star-rating/submit
                │
                ├─ Success → Thông báo thành công + update rating display
                │
                └─ Error → Hiển thị lỗi (đã đánh giá, bài viết không tồn tại, etc.)
```

#### Files

| File | Path | Description |
|------|------|-------------|
| `block.json` | `resources/blocks/review-form/block.json` | Block metadata |
| `edit.tsx` | `resources/blocks/review-form/edit.tsx` | Editor component |
| `save.tsx` | `resources/blocks/review-form/save.tsx` | Dynamic (returns null) |
| `render.php` | `resources/blocks/review-form/render.php` | Server-side render |
| `frontend.js` | `resources/blocks/review-form/frontend.js` | Frontend JS |
| `style.scss` | `resources/blocks/review-form/style.scss` | Frontend styles |
| `editor.scss` | `resources/blocks/review-form/editor.scss` | Editor styles |
| `index.tsx` | `resources/blocks/review-form/index.tsx` | Block registration |

#### PHP Config (render.php)

```php
wp_localize_script('jankx-review-form-frontend', 'jankxReviewForm', [
    'postId'        => $postId,
    'maxRating'     => 5,
    'showReview'    => true,
    'showProsCons'  => false,
    'showLoginForm' => true,
    'restUrl'       => 'https://nibitour.vn/wp-json/jankx/v1/star-rating/submit',
    'nonce'         => 'wp_rest_nonce',
    'isLoggedIn'    => false,
    'i18n'          => [
        'title'          => 'Đánh giá của bạn',
        'yourRating'     => 'Đánh giá của bạn',
        'submit'         => 'Gửi đánh giá',
        'submitting'     => 'Đang gửi...',
        'success'        => 'Cảm ơn bạn đã đánh giá!',
        'error'          => 'Có lỗi xảy ra, vui lòng thử lại.',
        'alreadyRated'   => 'Bạn đã đánh giá bài viết này rồi.',
        'loginRequired'  => 'Vui lòng đăng nhập để đánh giá.',
    ],
]);
```

#### Usage Example

```php
// Trang single-tour.php hoặc block template
<!-- Hiển thị rating hiện tại -->
<!-- wp:block {"name":"jankx/star-rating","attributes":{"ratingSource":"tour_rating","displayStyle":"summary","showCount":true}} /-->

<!-- Form đánh giá -->
<!-- wp:block {"name":"jankx/review-form","attributes":{"maxRating":5,"showProsCons":true}} /-->
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

Embed form đánh giá sao trên bất kỳ trang nào (alternative cho block).

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
        // Ví dụ: query từ external API, tính từ multiple sources, etc.
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
User submits rating (qua review-form block hoặc REST API)
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

### Core Framework (`jankx/includes/framework/Gutenberg/StarRating/`)

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

### Blocks (`resources/blocks/`)

| Block | Files |
|-------|-------|
| `star-rating` | `block.json`, `edit.tsx`, `index.tsx`, `style.scss`, `editor.scss` |
| `review-form` | `block.json`, `edit.tsx`, `save.tsx`, `render.php`, `frontend.js`, `style.scss`, `editor.scss` |

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

---

## Usage: Trang Tour Template

```php
<!-- single-tour.php hoặc block template -->

<!-- 1. Hiển thị rating hiện tại -->
<!-- wp:block {"name":"jankx/star-rating","attributes":{
    "ratingSource":"tour_rating",
    "displayStyle":"summary",
    "showCount":true,
    "starSize":18
}} /-->

<!-- 2. Form đánh giá -->
<!-- wp:block {"name":"jankx/review-form","attributes":{
    "maxRating":5,
    "showReview":true,
    "showProsCons":true,
    "formTitle":"Đánh giá Tour này",
    "submitText":"Gửi đánh giá"
}} /-->
```

Kết quả trên frontend:
```
┌─────────────────────────────────────────┐
│  ★ 4.2 (15 đánh giá)                    │  ← jankx/star-rating
├─────────────────────────────────────────┤
│  Đánh giá của bạn                       │
│  ┌─ ★ ★ ★ ★ ☆ ─┐                       │
│  │                                    │  ← jankx/review-form
│  │  [Nhận xét của bạn...]             │
│  │                                    │
│  │  Điểm mạnh      Điểm yếu           │
│  │  [...]          [...]              │
│  │                                    │
│  │  [Gửi đánh giá]                    │
│  └────────────────────────────────────┘
└─────────────────────────────────────────┘
```
