# Hibla

**A Fiber-native, enterprise-grade asynchronous ecosystem for PHP 8.4+**

[![Latest Release](https://img.shields.io/github/release/hiblaphp/hibla.svg?style=flat-square)](https://github.com/hiblaphp/hibla/releases)
[![PHP Version](https://img.shields.io/badge/PHP-8.4%2B-777bb4.svg?style=flat-square)](#)
[![MIT License](https://img.shields.io/badge/license-MIT-blue.svg?style=flat-square)](./LICENSE)

Hibla is a complete, bottom-up asynchronous stack designed exclusively for the modern PHP Fiber era. It provides a highly predictable, fault-tolerant, and memory-safe environment for building high-concurrency CLI applications, background workers, scrapers, and network daemons.

---

## Why Hibla?

The PHP ecosystem already has excellent async frameworks, but Hibla was built from scratch to explore architectural guarantees that are difficult to retrofit into older stacks:

1. **True Structured Concurrency:** Hibla introduces a strict 4-state Promise model (`pending`, `fulfilled`, `rejected`, and **`cancelled`**). If a parent operation fails or times out, all sibling operations are cleanly and synchronously cancelled. No "orphaned" promises are left dangling in memory.
2. **Enterprise Database Safety:** Cancelling an HTTP request is easy; cancelling a MySQL query is hard. Hibla's MySQL driver automatically opens a side-channel TCP connection to issue `KILL QUERY` when an operation is cancelled, releasing server-side table locks instantly. It also uses `COM_RESET_CONNECTION` to sanitize state between pooled requests.
3. **Erlang-Style Parallelism:** Asynchronous I/O cannot solve CPU-bound blocking. `hiblaphp/parallel` provides true multi-processing with a "Supervisor" architecture. If a worker process segfaults or hits an OOM error, the master process detects it, respawns the worker, and fires a recovery hook automatically.
4. **No Function Coloring:** You don't need to mark functions as `async`. You write plain PHP functions and use `await()`. The concurrency boundary lives entirely at the call site, keeping your domain logic clean.

### Addressing the "Fragmentation" Question

A common question when releasing a new tool is: *"Why not contribute to existing frameworks? Aren't you just fragmenting the async PHP ecosystem?"*

Hibla is not an attempt to fragment the ecosystem. It is an exploration of a **generational leap**. Existing frameworks carry the necessary weight of a decade of backward compatibility. Hibla is an experiment in starting fresh: *What if I built an entire ecosystem from the ground up, exclusively for PHP 8.4+, assuming native Fibers, strict typing, and property hooks from day one?*

I believe that exploring entirely new architectures, like out-of-band database cancellation and Erlang-style process supervision, creates healthy competition that pushes the entire PHP language forward. Choice is a sign of a maturing ecosystem.

---

## A First-Class Developer Experience

Performance and correctness are important, but they mean very little if the day-to-day experience of writing code feels like a chore. Hibla treats **Developer Experience (DX) as a first-class concern**, not an afterthought.

This is most visible in the areas where PHP developers spend most of their time:

**HTTP.** The `hiblaphp/http-client` is designed to feel immediately familiar. Making a request, streaming a response, handling Server-Sent Events, or wiring up interceptors should read cleanly and require as little boilerplate as possible. There is also a dedicated `hiblaphp/http-client-testing` package so that HTTP interactions in your application are fully simulatable in your test suite without spinning up a real server.

**Databases.** Working with MySQL through `hiblaphp/mysql` should feel as natural as any synchronous database library, with the added guarantee that connection pooling, statement caching, and cancellation are all handled for you at the driver level. You should not have to think about acquiring connections, killing orphaned queries, or resetting connection state between requests. Hibla handles all of that quietly in the background.

**Process Pools.** Spawning and managing worker processes for CPU-heavy work should not require you to understand Unix signals, pipe management, or IPC serialization at a deep level. `hiblaphp/parallel` exposes a simple, high-level API: define a pool size, submit work, and await results. The supervision, respawning, and error recovery all happen automatically.

**Async Primitives.** The building blocks of concurrency (`async`, `await`, `Promise`, `CancellationToken`, `Mutex`, `Semaphore`) are designed to compose naturally. You should be able to read a piece of concurrent Hibla code top-to-bottom and understand what it does without needing to trace through layers of callbacks or promise chains. If a piece of the API feels awkward or verbose, that is considered a bug worth fixing.

The goal is simple: async PHP should feel as productive and readable as the synchronous PHP you already know, just faster.

---

## Acknowledgements

Hibla did not emerge from a vacuum. It is the product of lessons learned from a decade of async programming across languages, runtimes, and libraries. I owe a significant intellectual debt to the communities and individuals who proved these ideas before I ever wrote a line of PHP.

I want to be completely honest: **Hibla would not exist without ReactPHP.**

ReactPHP did not merely inspire Hibla. It *proved* that the entire premise was possible. At a time when the PHP community largely dismissed the idea of a non-blocking, event-driven PHP runtime as a curiosity or an anti-pattern, the ReactPHP team built a complete, production-grade async ecosystem from the ground up. The event loop, the promise model, the stream abstraction, the DNS resolution pipeline, the socket layer... ReactPHP had all of it, years before native Fibers existed. The architecture of `hiblaphp/event-loop`, `hiblaphp/socket`, `hiblaphp/dns`, and `hiblaphp/stream` is directly and consciously modeled after the design decisions of the ReactPHP core team. I studied their code, internalized their tradeoffs, and built on top of the conceptual foundation they laid. If you use Hibla and it feels right, a large portion of that credit belongs to them.

**AmPHP** equally deserves deep acknowledgement, and not only for their async work. The structural Fiber concepts in `hiblaphp/async` were heavily shaped by AmPHP's pioneering work on coroutine-based concurrency in PHP. Their early exploration of generator-based async and their subsequent Fiber-first pivot gave the community an invaluable reference point. But beyond their library work, the Hibla project owes the AmPHP team a debt that extends to the PHP language itself. **Aaron Piotrowski** and **Niklas Keller**, both core AmPHP contributors, co-authored the [Fibers RFC (RFC 8208)](https://wiki.php.net/rfc/fibers) that landed in PHP 8.1. That RFC is not a footnote. It is the entire foundation that Hibla, and modern async PHP as a whole, is built on. The PHP community benefits from that work every single day, often without realizing where it came from. I realize it, and I want to say it plainly: **thank you, Aaron. Thank you, Niklas.**

Healthy ecosystems are built on standing on each other's shoulders. Hibla is my attempt to synthesize the best ideas from all of the above into something that feels native to modern PHP 8.4+. I genuinely encourage everyone using Hibla to star, contribute to, and support the ReactPHP and AmPHP projects.

---

## The Ecosystem

This `hiblaphp/hibla` meta-package requires the entire stack, allowing you to install everything with a single command. Alternatively, you can require individual components à la carte.

### Core Primitives

| Package | Description |
| --- | --- |
| [`hiblaphp/event-loop`](https://github.com/hiblaphp/event-loop) | Node.js-style phase-based event loop. Supports `ext-uv` (libuv) and pure PHP `stream_select`. |
| [`hiblaphp/promise`](https://github.com/hiblaphp/promise) | High-performance promises with strict cancellation, `Promise::all`, sliding-window concurrency, and mapping. |
| [`hiblaphp/async`](https://github.com/hiblaphp/async) | The `async()` and `await()` Fiber implementation. |
| [`hiblaphp/cancellation`](https://github.com/hiblaphp/cancellation) | `.NET`-style `CancellationToken` for propagating abort signals across independent workflows. |
| [`hiblaphp/sync`](https://github.com/hiblaphp/sync) | Async-aware `Mutex` and `Semaphore` for safely coordinating shared state across concurrent fibers. |

### Networking & I/O

| Package | Description |
| --- | --- |
| [`hiblaphp/stream`](https://github.com/hiblaphp/stream) | Non-blocking, event-driven streams with automatic backpressure handling. |
| [`hiblaphp/socket`](https://github.com/hiblaphp/socket) | Async TCP, TLS, and Unix sockets. Features mid-flight TLS upgrades and Happy Eyeballs (RFC 8305). |
| [`hiblaphp/dns`](https://github.com/hiblaphp/dns) | Non-blocking DNS resolution with caching, retry pipelines, and TCP fallback. |
| [`hiblaphp/http-client`](https://github.com/hiblaphp/http-client) | Fluent HTTP client with Server-Sent Events (SSE), streaming, interceptors, and robust proxy support. |
| [`hiblaphp/http-client-testing`](https://github.com/hiblaphp/http-client-testing) | Full HTTP request simulation framework for Pest/PHPUnit. |

### Database & State

| Package | Description |
| --- | --- |
| [`hiblaphp/sql`](https://github.com/hiblaphp/sql) | Common SQL contracts, connection interfaces, and strict isolation levels. |
| [`hiblaphp/mysql`](https://github.com/hiblaphp/mysql) | Pure-PHP MySQL binary protocol driver. Features lazy check-on-borrow pooling and deterministic LRU statement caching. |
| [`hiblaphp/cache`](https://github.com/hiblaphp/cache) | Promise-based cache abstraction with LRU array implementations and eviction hooks. |

### CPU Parallelism

| Package | Description |
| --- | --- |
| [`hiblaphp/parallel`](https://github.com/hiblaphp/parallel) | Process-based parallelism. Orchestrate self-healing worker pools for heavy CPU tasks safely alongside your async I/O. |

---

## Installation

Install the entire ecosystem:

```bash
composer require hiblaphp/hibla
```

**System Requirements:**

- PHP 8.4 or higher.
- `ext-curl` (Required for HTTP client).
- `ext-pcntl` (Recommended for graceful signal handling).
- `ext-uv` (Optional, but highly recommended for extreme-concurrency production environments).

---

## A Taste of Hibla (Synergy in Action)

Because all Hibla components share the exact same `Promise` and `CancellationToken` architecture, they compose flawlessly.

Here is an example of an application fetching data from a third-party API concurrently, processing the heavy CPU parsing in a supervised worker pool, and saving the results to MySQL, all constrained by a single 10-second timeout.

```php
use Hibla\Cancellation\CancellationTokenSource;
use Hibla\HttpClient\Http;
use Hibla\Mysql\MysqlClient;
use Hibla\Parallel\Parallel;
use Hibla\Promise\Promise;
use function Hibla\{async, await};

// 1. A single cancellation token to rule the entire workflow
$cts = new CancellationTokenSource(10.0); // Hard 10-second ceiling

// 2. Setup our ecosystem components
$db = new MysqlClient('mysql://user:pass@127.0.0.1/app');
$pool = Parallel::pool(size: 4)->boot(); // 4-core supervised worker pool

try {
    // 3. Fetch 100 pages from an API concurrently (max 10 at a time)
    $responses = await(
        Promise::map(range(1, 100), function ($page) use ($cts) {
            return Http::client()->get("https://api.example.com/data?page={$page}");
        }, concurrency: 10),
        $cts->token
    );

    // 4. Offload heavy CPU parsing to our parallel worker processes
    $parsedData = await(
        Promise::map($responses, function ($response) use ($pool) {
            return $pool->run(fn() => heavy_cpu_json_parsing($response->body()));
        }),
        $cts->token
    );

    // 5. Save to MySQL safely in a transaction
    await(
        $db->transaction(function ($tx) use ($parsedData) {
            foreach ($parsedData as $row) {
                await($tx->execute('INSERT INTO records (data) VALUES (?)', [$row]));
            }
        }),
        $cts->token
    );

    echo "Workflow completed successfully!";

} catch (\Hibla\Promise\Exceptions\CancelledException $e) {
    // If 10 seconds elapse:
    // - In-flight HTTP requests are aborted.
    // - Parallel workers are safely interrupted.
    // - The MySQL transaction is automatically rolled back.
    // - Orphaned MySQL queries are killed via side-channel.
    echo "Workflow timed out. Everything cleaned up safely.";
}

$pool->shutdown();
```

---

### Performance & The Road to an HTTP Server

Because Hibla uses `SO_REUSEPORT` combined with Erlang-style `Parallel` worker pools, the OS kernel natively load-balances incoming TCP connections across all available CPU cores.

To test the absolute ceiling of the Hibla Socket and Event Loop engine, a raw, bare-bones HTTP responder was put together.

![Hibla Benchmark showing 116,000 RPS](arts/benchmark.png)

**How it works (in ~30 lines of code):**

```php
<?php

declare(strict_types=1);

require __DIR__ . '/../vendor/autoload.php';
require __DIR__ . '/sample_router.php';

use Hibla\Parallel\Parallel;
use Hibla\Socket\SocketServer;

$poolSize = 8;
$pool = Parallel::pool(size: $poolSize)->withoutTimeout()->boot();

for ($i = 0; $i < $poolSize; $i++) {
    $pool->run(function () {
        $server = new SocketServer('127.0.0.1:8080', [
            'tcp' => ['so_reuseport' => true, 'backlog' => 65535],
        ]);

        $server->on('connection', function ($connection) {
            $connection->on('data', function (string $rawRequest) use ($connection) {

                $response = "HTTP/1.1 200 OK\r\n"
                    . "Content-Type: text/plain\r\n"
                    . "Content-Length: 13\r\n"
                    . "Connection: keep-alive\r\n\r\n"
                    . "Hello, Hibla!";

                $connection->write($response);
            });
        });
    });
}
```

#### An Honest Note on Benchmarks

*Is Hibla actually going to serve 116,000 RPS in production?* **No, or maybe yes.**

This benchmark is a raw TCP socket returning a hardcoded string. It bypasses proper HTTP header parsing, chunked encoding validation, PSR-7 object hydration, and routing logic.

This benchmark was run to prove one thing: **the base engine has virtually zero overhead.** The Event Loop and the IPC (Inter-Process Communication) layer are rock-solid. When the official `hiblaphp/http-server` lands in Phase 2, the RPS will naturally be lower due to the demands of strict HTTP protocol compliance, but the starting point is an incredibly fast, non-blocking foundation.

---

## Current Status & Roadmap

The Hibla ecosystem is currently in **Public Beta**.

The architecture is stable, heavily stress-tested, and currently being dogfooded in production environments. I am gathering community feedback on the DX (Developer Experience) and API ergonomics before tagging a stable `1.0.0` release.

### What's Next (Phase 2)?

- **`hiblaphp/postgres`**: A native PostgreSQL binary protocol driver mirroring the MySQL architecture, with the same out-of-band `pg_cancel_backend()` cancellation guarantee and lazy check-on-borrow connection pooling.
- **`hiblaphp/http-server`**: A Fiber-native web server built directly on the Hibla Engine to serve HTTP/1.1 and HTTP/2 traffic without relying on external PHP SAPIs.

### On the Horizon (Phase 3)

- **`hiblaphp/redis`**: A pure-PHP, non-blocking Redis client implementing the RESP3 protocol over the Hibla socket layer. It will support connection pooling, pipelining, Pub/Sub channels as async event emitters, and full `CancellationToken` propagation, bringing the same structured concurrency guarantees already found in `hiblaphp/mysql` to Redis workflows.
- **`hiblaphp/websocket`**: A full-duplex, Fiber-native WebSocket implementation (RFC 6455) covering both client and server roles. The server side will integrate directly with `hiblaphp/http-server` for seamless HTTP-to-WebSocket upgrade handling, while the client will be built on top of `hiblaphp/http-client` and `hiblaphp/socket`. Planned features include per-message compression (RFC 7692), configurable frame fragmentation, automatic ping/pong heartbeating, and first-class `CancellationToken` support for graceful teardown.

---

## Contributing & Community

Contributions, issues, and feature requests are highly welcome.

Because Hibla relies heavily on strict types, property hooks, and PHP 8.4 features, all PRs must pass `phpstan` at `level: max` and adhere to strict PSR-12 formatting via Laravel Pint. See the `CONTRIBUTING.md` in any individual repository for details.

If you find a bug or have an architectural suggestion, please open an issue in the specific repository (for example, `hiblaphp/mysql` or `hiblaphp/event-loop`).

---

## Author

Hibla is designed, built, and maintained by **Reymart A. Calicdan**.

If you have questions, ideas, want to be a maintainer, or just want to talk async PHP, feel free to reach out through any of the channels below:

- Email: [reymart.calicdan06@gmail.com](mailto:reymart.calicdan06@gmail.com)
- X: [@rcalicdan06](https://x.com/rcalicdan06)
- Mastodon: [@rcalicdan@phpc.social](https://phpc.social/@rcalicdan)
- Reddit: [u/rcalicdan](https://www.reddit.com/user/rcalicdan/)
---

## Support & Sponsorship

Hibla is a passion project built and maintained in my own time. It is completely free and open source, and I intend to keep it that way. That said, building and maintaining a full async ecosystem is a significant ongoing effort, and any support goes a long way toward keeping it alive, well-documented, and actively developed.

If Hibla has saved you time, helped your team ship something, or simply sparked your interest in async PHP, please consider supporting the project.

- **GitHub Sponsors (Personal):** [github.com/sponsors/rcalicdan](https://github.com/sponsors/rcalicdan)
- **GitHub Sponsors (Hibla):** [github.com/sponsors/hiblaphp](https://github.com/sponsors/hiblaphp)

Every contribution, no matter the size, is genuinely appreciated.

---

### Sponsors

Hibla is proudly supported by the following companies and individuals. If your company uses Hibla in production or wants to support the async PHP ecosystem, sponsoring the project is a great way to get visibility while directly funding its development. Reach out at [reymart.calicdan06@gmail.com](mailto:reymart.calicdan06@gmail.com) to discuss sponsorship tiers.

### Sponsors

Sponsoring Hibla is a great way to get visibility for your company while directly funding the async PHP ecosystem. Reach out at [reymart.calicdan06@gmail.com](mailto:reymart.calicdan06@gmail.com) to discuss sponsorship tiers.

#### Gold Sponsors

*No gold sponsors yet. [Your logo could be here.](mailto:reymart.calicdan06@gmail.com)*

#### Silver Sponsors

*No silver sponsors yet. [Your logo could be here.](mailto:reymart.calicdan06@gmail.com)*

#### Community Backers

*No community backers yet. [Your logo could be here.](mailto:reymart.calicdan06@gmail.com)*

## License

The Hibla Ecosystem is open-sourced software licensed under the [MIT license](https://opensource.org/licenses/MIT).
