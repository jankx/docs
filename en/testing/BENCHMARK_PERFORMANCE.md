# Benchmark & Performance

Performance tooling and measured baseline for the Jankx theme framework, the
Nobitour child theme, and the full extension ecosystem.

## Why this exists

Wall-clock timing alone cannot tell you *why* a page is slow. A 10 second page
load could be one slow query, or a million cheap function calls. This harness
measures both:

- **Wall time** - what a visitor waits (HTTP, and PHP stage timings).
- **Call graph** - which functions are called, how often, and where the CPU goes.
- **Database** - query count, SQL time, and structurally duplicated queries.
- **Payload** - HTML size, autoloaded options, included PHP files, memory.

## Tooling

| File | Purpose |
| --- | --- |
| `bench.php` | CLI entry point: `probe`, `calls`, `trace`, `http`, `report`, `graph` |
| `collect.php` | Boots WordPress inside a child PHP process and emits JSON |
| `lib/CallProfiler.php` | Spawns the child process under Xdebug, locates output files |
| `lib/CachegrindParser.php` | Parses Xdebug cachegrind output into a call graph |
| `lib/FlameGraph.php` | Renders the parsed call graph as a standalone SVG/HTML flame graph |
| `lib/analyze_existing.php` | Analyses an existing cachegrind capture, no server needed |
| `lib/who_calls.php` | Shows which functions call a given hotspot |
| `lib/find_duplicate_queries.php` | Finds repeated SQL and attributes it to a real call site |

Artifacts are written to `benchmarks/.work/` and must not be committed.

### Commands

Run from the `jankx` theme directory:

```bash
# WordPress probe only, no profiler overhead.
php benchmarks/bench.php probe --scenario=boot

# Probe with a full front-page render.
php benchmarks/bench.php probe --scenario=home

# Xdebug profile + parsed call graph (the main analysis command).
php benchmarks/bench.php calls --scenario=home --top=30

# Xdebug trace: flat function entry counts (cheaper to read than a profile).
php benchmarks/bench.php trace --scenario=boot --top=25

# End-to-end HTTP latency, plus cache/compression header check.
php benchmarks/bench.php http --runs=5

# Write a JSON report for CI or before/after comparison.
php benchmarks/bench.php report --scenario=home

# Interactive flame graph (HTML + SVG in benchmarks/report/).
php benchmarks/bench.php graph --scenario=home

# Re-render an existing capture instead of profiling again.
php benchmarks/bench.php graph --scenario=home --file=benchmarks/.work/cachegrind_home/jankx_home_28724
```

`--scenario` accepts `boot`, `home`, and `page`. `boot` stops after WordPress
finishes loading; `home` and `page` additionally render the front template.

`--width` sets the SVG pixel width, `--depth` caps how many levels are drawn
(both default: width 1400, depth 14).

### Reading the flame graph

`graph` writes a self-contained HTML file into `benchmarks/report/` - open it
directly in a browser, no server needed. Hovering a frame shows the function
name, its self time, and its call count.

Two properties make the widths worth trusting:

- Every timed function gets exactly one frame, sized by the self time of its own
  body. Cachegrind aggregates edges by function name, which makes the call graph
  a DAG rather than a tree; the renderer builds a spanning tree so no function is
  counted twice.
- The drawn self times therefore sum exactly to the profiler total. If the header
  `total inclusive` does not match `calls`' `profiled CPU`, the graph is broken.

Frame width is the subtree's self time, not a single function's inclusive cost.
A classic stack flame graph is not reproducible from cachegrind data, because the
capture has no stack ordering.

### Xdebug

The harness runs its own child processes, so `xdebug.mode` stays `off` in
`php.ini` and never slows normal traffic. The child process is invoked with
`xdebug.mode=profile` (or `trace`) explicitly.

Two format details the parser handles, both of which silently corrupt results if
ignored:

- Xdebug writes `events: Time_(10ns)`. Raw cost values are 10-nanosecond ticks,
  not microseconds. Assuming microseconds inflates every duration 100x.
- Xdebug writes `positions: line`, which prepends a source-line column to each
  cost line. Reading that column as the time cost inflates durations again.

`cost unit` is printed in the `calls` output so the conversion is verifiable.
Always sanity-check it against the reported `WP wall time`.

## Diagnostic tools

Three scripts in `benchmarks/lib/` reuse the same parser as `bench.php` but
split the investigation into separate steps. They complement `calls` rather
than replace it: `calls` answers "where does the time go", these answer "who
called it" and "which SQL is wasted".

### Analyse an existing capture (no server required)

`bench.php calls` always re-runs the collector, so it needs Xdebug and a live
web server. Once cachegrind files exist under `benchmarks/.work/`, they can be
re-analysed offline:

```bash
# Every file under .work/cachegrind_*
php benchmarks/lib/analyze_existing.php

# One specific file
php benchmarks/lib/analyze_existing.php benchmarks/.work/cachegrind_home/jankx_home_23280
```

It prints three blocks: top self time, top call count, and DB/cache-related
functions. A home-page capture is around 90 MB, so each run takes 20-30 seconds
to parse.

### Find what calls a hotspot

Cachegrind is unreliable about a function's file: when Xdebug reuses a name id,
the file attribute drifts to the *caller*'s context.
`WP_Scripts::get_highest_fetchpriority_with_dependents`, for example, gets
stamped with the name of a completely unrelated class. Walk the call graph
instead of reading that column:

```bash
php benchmarks/lib/who_calls.php benchmarks/.work/cachegrind_home/jankx_home_23280
```

It prints up to 8 direct callers for each hotspot in a fixed target list
(including `apply_filters`, `get_option`, `wpdb->query`, `WP_Query->query`, and
so on).

### Find repeated SQL

This one needs no Xdebug, only `SAVEQUERIES`:

```bash
php benchmarks/lib/find_duplicate_queries.php

# Inspect the raw structure of any logged query
JANKX_DEBUG_ROW=120 php benchmarks/lib/find_duplicate_queries.php
```

Run it from the theme directory; no server and no Xdebug needed. It builds a
browser-like request context, boots WordPress, renders the front template, then
reads `$wpdb->queries`.

The important part: the output is split into two groups, and reading them
correctly is the whole point of the tool.

- **Exactly repeated** - the identical SQL statement ran multiple times. This is
  genuine waste and this is what you should go fix.
- **Same-shape different-params** - the same statement shape with different
  parameters (different IDs/terms). This is normal batched access, *not* waste.
  The "redundant ms" column for this group is just volume, not time you can
  save.

Do not trust the "redundant" figure printed by `collect.php`: it collapses
literals, so 52 distinct options get merged into one group and the number
overshoots. Use the caller column this script prints instead.

The caller comes from the call-chain string `wpdb` stores in
`$wpdb->queries[n][2]`, so it is a string of function names rather than a
backtrace array. Only frames matching `Jankx` or `App\` are labelled as theme
code; everything else shows as `(core only)`.

### Suggested investigation loop

1. `bench.php probe --scenario=home` for the headline numbers.
2. `analyze_existing.php` on any existing capture for self time and call counts.
3. `who_calls.php` to find where the hotspot is invoked.
4. `find_duplicate_queries.php` for the SQL side.
5. Fix, then re-run `probe` and compare. **Always check the HTML size before and
   after** - if the output changed you broke a feature, however few queries
   remain.

Do not optimise by call count while Xdebug is on: per-call overhead is inflated
enough that a function called 17,000 times still costs single-digit
milliseconds in production. Prioritise `curl_exec`, raw SQL counts, and any DB
writes happening during render - those are not distorted by the profiler.

## Baseline

Measured on the local stack: nginx 1.28.2, PHP 8.3.30 (ZTS), MySQL 8.4,
WordPress with Debug Bar and Polylang active, `WP_DEBUG` enabled.

> Profiler numbers include Xdebug's own overhead. Treat them as relative
> signals for ranking, not as production latency.

### HTTP (`--runs=3`)

| Metric | Value |
| --- | --- |
| Cold | 1329 ms |
| Median (warm) | 996 ms |
| Min | 954 ms |
| HTML | 710.2 KB |
| `Content-Encoding` | **absent** - HTML sent uncompressed |
| `Cache-Control` / `Expires` | absent |
| `Server` | nginx/1.28.2 |

### PHP stages (`--scenario=home`)

| Stage | Time |
| --- | --- |
| `wp_loaded` | 3444.7 ms |
| `query_parsed` | 3479.2 ms |
| `rendered` (total) | 10307 ms |

Render dominates: roughly 6.8 seconds are spent after WordPress has loaded and
parsed the query.

### Call profile (`--scenario=home`)

| Metric | Value |
| --- | --- |
| Functions profiled | 5,577 |
| Total call events | 1,307,002 |
| Profiled CPU time | 11.28 s |
| Cost unit | 10 ns |

The `graph` command reports the same 11,602.72 ms as `total inclusive`, with 8
root children and all 5,577 timed functions drawn exactly once.

Top self-time functions:

| Self time | Calls | Function |
| --- | --- | --- |
| 654.5 ms | 1 | `curl_exec` (external HTTP call during render) |
| 641.4 ms | 1 | `require_once wp-settings.php` |
| 462.9 ms | 59,801 | `apply_filters` |
| 407.8 ms | 560 | Polylang Composer ClassLoader closure |
| 349.6 ms | 7,092 | `WP_Hook->apply_filters` |
| 345.3 ms | 35,127 | `_wp_array_get` |
| 336.2 ms | 82 | `WP_Theme_JSON::sanitize` |
| 294.7 ms | 10,950 | `WP_HTML_Tag_Processor->parse_next_attribute` |
| 256.4 ms | 17,535 | `WP_Scripts->get_highest_fetchpriority_with_dependents` |

Notable amplification edges:

| Calls | Edge |
| --- | --- |
| 25,431 | `ClassLoader->findFileWithExtension` -> `substr` |
| 25,172 | `WP_Scripts->get_dependents` -> `in_array` |
| 24,116 | `WP_Theme_JSON::sanitize` -> `array_keys` |
| 17,753 | `WP_Theme_JSON->merge` -> `_wp_array_get` (211.9 ms inclusive) |
| 17,474 | `WP_Scripts->get_highest_fetchpriority_with_dependents` (recursive, 913.9 ms) |
| 11,922 | `get_option` -> `apply_filters` |

### Trace (`--scenario=boot`)

307,593 entries. Boot-time leaders: `substr` 25,916, `apply_filters` 25,392,
`strrpos` 19,918, Composer `loadClass`/`findFile` 4,148 each. Autoloading and
hook dispatch dominate boot.

### Database (`--scenario=home`)

355 queries, 242 ms of SQL time, 22 duplicate groups accounting for 302
redundant queries.

| Table | Queries |
| --- | --- |
| `wp_posts` | 125 |
| `wp_options` | 76 |
| `wp_terms` | 71 |
| `wp_postmeta` | 56 |

Top duplicated query shapes:

| Count | Shape |
| --- | --- |
| 52 | `SELECT option_value FROM wp_options WHERE option_name = ?` |
| 51 | `SELECT ... FROM wp_postmeta WHERE post_id IN (...)` |
| 46 | `SELECT * FROM wp_posts WHERE ID = ?` |
| 42 | Taxonomy `SQL_CALC_FOUND_ROWS` query |
| 22 | `SELECT DISTINCT t.term_id ...` |
| 17 | `SELECT FOUND_ROWS()` |

### Payload

| Metric | Value |
| --- | --- |
| Autoloaded options | 236 (88.3 KB) |
| Included PHP files | 1,467 |
| Block types registered | 272 (86 server-rendered) |
| Hooks / callbacks | 841 / 1,681 |
| Peak memory | 24 MB |

## Findings and recommendations

Ranked by expected impact on the numbers above.

### 1. Enable compression and cache headers

The single cheapest win: 710 KB of HTML is served uncompressed with no
`Cache-Control`. Enabling gzip/brotli typically cuts HTML size by 70-80% and is
worth more than any PHP-level change on this page.

### 2. Cache `get_option` lookups

`apply_filters` runs 59,801 times and `get_option` alone issues 52 identical
`wp_options` lookups. Introducing a persistent object cache (Redis/Memcached)
would collapse both `apply_filters` volume and the 302 redundant queries.

### 3. Investigate `curl_exec` during render

654 ms of the profile is a single `curl_exec` on the render path. If it fetches
remote content that is not essential to first paint, move it to an async
request or cache the response.

### 4. Reduce `WP_Theme_JSON` work

`WP_Theme_JSON::sanitize` and `->merge` together account for ~490 ms of self
time, driven by 82 sanitize calls and a 17,753-call edge into `_wp_array_get`.
Theme JSON resolution is being repeated far more than necessary; caching or
pre-compiling theme JSON data would cut this.

### 5. Batch or cache `wp_posts` by ID

`SELECT * FROM wp_posts WHERE ID = ?` runs 46 times. A primed object cache
(`wp_prime_option_caches` / post cache warming) removes this almost entirely.

### 6. Set a persistent object cache as a project requirement

Every item above compounds. With Redis in place, items 2 and 5 largely resolve
together and the remaining PHP work becomes the only variable.

## Caveats

- Numbers include `WP_DEBUG`, Debug Bar, and Polylang, all of which add
  measurable overhead. Re-measure with them disabled before treating the
  absolute values as production latency.
- Xdebug inflates totals. Use `http --runs=N` for latency and `calls` for
  relative hotspot ranking.
- `php:internal` functions such as `substr` and `in_array` are mostly noise
  unless they are on a hot edge, as several are here.
- Only the local nginx stack was measured. See the LiteSpeed section before
  extrapolating server-level caching behaviour.

## LiteSpeed

No LiteSpeed-specific benchmark has been run. The environment used nginx, and
`litespeed-cache` is not installed.

To benchmark LiteSpeed properly:

1. Migrate to OpenLiteSpeed (or run it alongside) - LiteSpeed page cache and ESI
   are server features and will not engage under nginx.
2. Install `litespeed-cache` and confirm the cache hit/miss indicator appears
   in the response headers.
3. Re-run `php benchmarks/bench.php http --runs=5` and compare the median against
   the 996 ms nginx baseline recorded above.
4. Re-check the `CACHE / COMPRESSION HEADERS` section: a warm LiteSpeed hit
   should report caching headers that nginx did not.

Until that is done, treat all LiteSpeed claims here as untested. Installing
`litespeed-cache` under nginx does not give full page caching.