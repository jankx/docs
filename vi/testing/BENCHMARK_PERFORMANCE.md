# Benchmark & Hiệu Năng

Công cụ đo hiệu năng và baseline đã đo cho Jankx theme framework, child theme
Nobitour, và toàn bộ hệ sinh thái extension.

## Mục đích

Chỉ đo wall-clock không cho biết *tại sao* một trang chậm. Trang tải 10 giây
có thể do một truy vấn chậm, hoặc do hơn một triệu lời gọi hàm rẻ tiền. Bộ công
cụ này đo cả hai:

- **Wall time** - thời gian người dùng phải chờ (HTTP và các giai đoạn PHP).
- **Call graph** - hàm nào được gọi, gọi bao nhiêu lần, CPU nằm ở đâu.
- **Cơ sở dữ liệu** - số truy vấn, thời gian SQL, và truy vấn trùng cấu trúc.
- **Payload** - kích thước HTML, options autoload, số file PHP nạp vào, memory.

## Công cụ

| File | Vai trò |
| --- | --- |
| `bench.php` | CLI: `probe`, `calls`, `trace`, `http`, `report`, `graph` |
| `collect.php` | Khởi động WordPress trong tiến trình PHP con và xuất JSON |
| `lib/CallProfiler.php` | Chạy tiến trình con dưới Xdebug, tìm file output |
| `lib/CachegrindParser.php` | Phân tích output cachegrind của Xdebug thành call graph |
| `lib/FlameGraph.php` | Vẽ call graph đã parse thành flame graph SVG/HTML độc lập |

Artifact được ghi vào `benchmarks/.work/` và không được commit.

### Các lệnh

Chạy từ thư mục theme `jankx`:

```bash
# Chỉ probe WordPress, không có overhead profiler.
php benchmarks/bench.php probe --scenario=boot

# Probe kèm render trang chủ.
php benchmarks/bench.php probe --scenario=home

# Xdebug profile + call graph đã parse (lệnh phân tích chính).
php benchmarks/bench.php calls --scenario=home --top=30

# Xdebug trace: đếm số lần gọi hàm (dễ đọc hơn profile).
php benchmarks/bench.php trace --scenario=boot --top=25

# Đo latency HTTP end-to-end, kèm kiểm tra header cache/nén.
php benchmarks/bench.php http --runs=5

# Xuất báo cáo JSON để CI hoặc so sánh trước/sau.
php benchmarks/bench.php report --scenario=home

# Flame graph tương tác (HTML + SVG trong benchmarks/report/).
php benchmarks/bench.php graph --scenario=home

# Vẽ lại từ capture đã có thay vì profile lại.
php benchmarks/bench.php graph --scenario=home --file=benchmarks/.work/cachegrind_home/jankx_home_28724
```

`--scenario` nhận `boot`, `home`, `page`. `boot` dừng sau khi WordPress nạp
xong; `home` và `page` render thêm front template.

`--width` đặt chiều rộng pixel của SVG, `--depth` giới hạn số tầng được vẽ
(mặc định: width 1400, depth 14).

### Đọc flame graph

Lệnh `graph` ghi ra một file HTML độc lập trong `benchmarks/report/` - mở
trực tiếp bằng trình duyệt, không cần server. Rê chuột lên một frame để xem
tên hàm, self time và số lần gọi.

Hai tính chất làm cho độ rộng frame đáng tin cậy:

- Mỗi hàm có thời gian thực sự chỉ nhận đúng một frame, kích thước theo self time
  của thân hàm. Cachegrind gộp edge theo tên hàm, nên call graph là DAG chứ không
  phải cây; bộ vẽ dựng một spanning tree để không hàm nào bị đếm hai lần.
- Vì vậy tổng self time được vẽ luôn bằng đúng tổng của profiler. Nếu
  `total inclusive` ở header không khớp `profiled CPU` của lệnh `calls`, graph đó
  đang sai.

Độ rộng frame là self time của cả subtree, không phải inclusive cost của một hàm
đơn lẻ. Flame graph kiểu stack cổ điển không thể tái tạo từ dữ liệu cachegrind,
vì capture không lưu thứ tự stack.

### Xdebug

Harness tự chạy tiến trình con, nên `xdebug.mode` để `off` trong `php.ini` và
không bao giờ làm chậm traffic thường. Tiến trình con được gọi với
`xdebug.mode=profile` (hoặc `trace`) một cách tường minh.

Hai chi tiết định dạng mà parser xử lý, cả hai đều làm hỏng kết quả nếu bỏ qua:

- Xdebug ghi `events: Time_(10ns)`. Giá trị cost thô tính bằng tick 10 nano-giây,
  không phải micro-giây. Giả định micro-giây làm mọi thời gian lớn hơn 100 lần.
- Xdebug ghi `positions: line`, thêm một cột số dòng ở đầu mỗi cost line. Đọc cột
  đó như cost thời gian lại làm thời gian phình lên lần nữa.

`cost unit` được in ra trong output của `calls` để có thể kiểm chứng phép đổi
đơn vị. Luôn đối chiếu với `WP wall time` đã báo.

## Baseline

Đo trên stack local: nginx 1.28.2, PHP 8.3.30 (ZTS), MySQL 8.4, WordPress có
Debug Bar và Polylang đang bật, `WP_DEBUG` bật.

> Số liệu có profiler đã bao gồm overhead của chính Xdebug. Hãy hiểu chúng là
> tín hiệu tương đối để xếp hạng, không phải latency production.

### HTTP (`--runs=3`)

| Chỉ số | Giá trị |
| --- | --- |
| Cold | 1329 ms |
| Median (warm) | 996 ms |
| Min | 954 ms |
| HTML | 710.2 KB |
| `Content-Encoding` | **không có** - HTML gửi không nén |
| `Cache-Control` / `Expires` | không có |
| `Server` | nginx/1.28.2 |

### Các giai đoạn PHP (`--scenario=home`)

| Giai đoạn | Thời gian |
| --- | --- |
| `wp_loaded` | 3444.7 ms |
| `query_parsed` | 3479.2 ms |
| `rendered` (tổng) | 10307 ms |

Render chiếm phần lớn: khoảng 6.8 giây được tiêu sau khi WordPress đã nạp và
parse query.

### Call profile (`--scenario=home`)

| Chỉ số | Giá trị |
| --- | --- |
| Số hàm được profile | 5,577 |
| Tổng số call event | 1,307,002 |
| CPU time được profile | 11.28 s |
| Đơn vị cost | 10 ns |

Lệnh `graph` báo đúng 11,602.72 ms ở `total inclusive`, với 8 root child và
toàn bộ 5,577 hàm có thời gian được vẽ đúng một lần.

Top hàm theo self time:

| Self time | Số lần gọi | Hàm |
| --- | --- | --- |
| 654.5 ms | 1 | `curl_exec` (gọi HTTP ngoài trong lúc render) |
| 641.4 ms | 1 | `require_once wp-settings.php` |
| 462.9 ms | 59,801 | `apply_filters` |
| 407.8 ms | 560 | Polylang Composer ClassLoader closure |
| 349.6 ms | 7,092 | `WP_Hook->apply_filters` |
| 345.3 ms | 35,127 | `_wp_array_get` |
| 336.2 ms | 82 | `WP_Theme_JSON::sanitize` |
| 294.7 ms | 10,950 | `WP_HTML_Tag_Processor->parse_next_attribute` |
| 256.4 ms | 17,535 | `WP_Scripts->get_highest_fetchpriority_with_dependents` |

Các cạnh khuếch đại đáng chú ý:

| Số lần | Cạnh |
| --- | --- |
| 25,431 | `ClassLoader->findFileWithExtension` -> `substr` |
| 25,172 | `WP_Scripts->get_dependents` -> `in_array` |
| 24,116 | `WP_Theme_JSON::sanitize` -> `array_keys` |
| 17,753 | `WP_Theme_JSON::merge` -> `_wp_array_get` (211.9 ms inclusive) |
| 17,474 | `WP_Scripts->get_highest_fetchpriority_with_dependents` (đệ quy, 913.9 ms) |
| 11,922 | `get_option` -> `apply_filters` |

### Trace (`--scenario=boot`)

307,593 entry. Các hàm dẫn đầu lúc boot: `substr` 25,916, `apply_filters`
25,392, `strrpos` 19,918, Composer `loadClass`/`findFile` mỗi 4,148. Autoload và
dispatch hook chi phối thời gian boot.

### Cơ sở dữ liệu (`--scenario=home`)

355 truy vấn, 242 ms SQL, 22 nhóm trùng lặp chiếm 302 truy vấn thừa.

| Bảng | Số truy vấn |
| --- | --- |
| `wp_posts` | 125 |
| `wp_options` | 76 |
| `wp_terms` | 71 |
| `wp_postmeta` | 56 |

Top truy vấn trùng cấu trúc:

| Số lần | Dạng truy vấn |
| --- | --- |
| 52 | `SELECT option_value FROM wp_options WHERE option_name = ?` |
| 51 | `SELECT ... FROM wp_postmeta WHERE post_id IN (...)` |
| 46 | `SELECT * FROM wp_posts WHERE ID = ?` |
| 42 | Taxonomy `SQL_CALC_FOUND_ROWS` |
| 22 | `SELECT DISTINCT t.term_id ...` |
| 17 | `SELECT FOUND_ROWS()` |

### Payload

| Chỉ số | Giá trị |
| --- | --- |
| Options autoload | 236 (88.3 KB) |
| File PHP được nạp | 1,467 |
| Block type đã đăng ký | 272 (86 server-rendered) |
| Hook / callback | 841 / 1,681 |
| Peak memory | 24 MB |

## Phát hiện và khuyến nghị

Xếp theo mức tác động dự kiến lên các số liệu trên.

### 1. Bật nén và header cache

Chiến thắng rẻ nhất: 710 KB HTML được gửi không nén và không có `Cache-Control`.
Bật gzip/brotli thường giảm 70-80% kích thước HTML, giá trị này lớn hơn mọi
thay đổi ở tầng PHP với trang này.

### 2. Cache kết quả `get_option`

`apply_filters` chạy 59,801 lần và riêng `get_option` đã phát ra 52 truy vấn
`wp_options` giống hệt nhau. Object cache bền vững (Redis/Memcached) sẽ loại bỏ
cả khối lượng `apply_filters` lẫn 302 truy vấn thừa.

### 3. Kiểm tra `curl_exec` trong lúc render

654 ms trong profile là một lời gọi `curl_exec` duy nhất trên đường render. Nếu
nó tải nội dung từ xa không thiết yếu cho lần hiển thị đầu, nên chuyển sang
request bất đồng bộ hoặc cache lại response đó.

### 4. Giảm khối lượng xử lý `WP_Theme_JSON`

`WP_Theme_JSON::sanitize` và `->merge` cộng lại khoảng 490 ms self time, kéo
theo bởi 82 lần sanitize và cạnh 17,753 lần gọi vào `_wp_array_get`. Việc resolve
theme JSON đang lặp lại nhiều hơn cần thiết; cache hoặc pre-compile dữ liệu theme
JSON sẽ cắt đáng kể phần này.

### 5. Gộp hoặc cache `wp_posts` theo ID

`SELECT * FROM wp_posts WHERE ID = ?` chạy 46 lần. Object cache dạng persistent
(warm post cache) loại bỏ gần như toàn bộ.

### 6. Đặt object cache bền vững là yêu cầu của dự án

Các mục trên cộng dồn. Khi có Redis, mục 2 và 5 cùng được giải quyết, và phần
PHP còn lại trở thành biến duy nhất cần theo dõi.

## Lưu ý

- Số liệu bao gồm `WP_DEBUG`, Debug Bar và Polylang, đều tạo overhead đáng kể.
  Hãy đo lại khi tắt chúng trước khi coi giá trị tuyệt đối là latency production.
- Xdebug làm phình tổng thời gian. Dùng `http --runs=N` để đo latency và `calls`
  để xếp hạng hotspot tương đối.
- Hàm `php:internal` như `substr` và `in_array` chủ yếu là nhiễu, trừ khi chúng
  nằm trên một cạnh nóng như một số trường hợp ở đây.
- Chỉ mới đo trên nginx local. Xem phần LiteSpeed trước khi suy rộng hành vi
  cache ở tầng server.

## LiteSpeed

Chưa chạy benchmark nào riêng cho LiteSpeed. Môi trường đo dùng nginx và
`litespeed-cache` chưa được cài.

Để benchmark LiteSpeed đúng cách:

1. Chuyển sang OpenLiteSpeed (hoặc chạy song song) - page cache và ESI của
   LiteSpeed là tính năng ở tầng server và sẽ không hoạt động dưới nginx.
2. Cài `litespeed-cache` và xác nhận chỉ báo cache hit/miss xuất hiện trong
   response header.
3. Chạy lại `php benchmarks/bench.php http --runs=5` và so median với baseline
   nginx 996 ms đã ghi ở trên.
4. Kiểm tra lại phần `CACHE / COMPRESSION HEADERS`: một lần hit LiteSpeed ấm
   phải trả về các header cache mà nginx không có.

Cho tới khi làm được điều đó, mọi khẳng định về LiteSpeed trong tài liệu này đều
chưa được kiểm chứng. Cài `litespeed-cache` dưới nginx không mang lại page cache
đầy đủ.