# raven-log

[![CI](https://github.com/martian56/raven-log/actions/workflows/ci.yml/badge.svg)](https://github.com/martian56/raven-log/actions/workflows/ci.yml)

A leveled logger for Raven. Four levels, a minimum-level filter, colored tags
when you want them, and an optional name to mark which part of the app a line
came from. It's just `lib.rv`, no dependencies.

## Install

```toml
[dependencies]
"github.com/martian56/raven-log" = "v0.1.0"
```

## Usage

For a quick script, the package-level functions log through a shared logger set
to Info:

```rust
import "github.com/martian56/raven-log" { info, warn, error }

fun main() {
    info("server started")
    warn("disk almost full")
    error("connection refused")
}
```

When you want your own settings, build a `Logger` and chain the configuration.
Each call hands back a configured copy, so a `Logger` is just a value you can
pass around:

```rust
import "github.com/martian56/raven-log" { Logger, Level }

fun main() {
    let log = Logger.new().named("api").level(Level.Debug)
    log.debug("handling GET /users")
    log.info("200 OK in 4ms")

    // No color, and only Warn and above:
    let q = Logger.new().colored(false).level(Level.Warn).named("worker")
    q.info("this one is dropped")
    q.warn("retrying job 42")
}
```

Lines go to standard output as `[LEVEL] name: message`. The name and its colon
are left off when the logger doesn't have one, and the level tag is wrapped in
an ANSI color unless you turn that off with `.colored(false)`, which is what you
want for a log file.

## Levels

`Debug`, `Info`, `Warn`, `Error`, in that order. A logger drops anything below
its minimum, so a `Warn` logger stays quiet about `Debug` and `Info`.

## API

### `Logger`

| Method | Description |
|--------|-------------|
| `Logger.new()` | A logger at `Info`, unnamed, colored |
| `log.level(min)` | A copy with a new minimum level |
| `log.named(name)` | A copy tagged with `name` |
| `log.colored(on)` | A copy with color on or off |
| `log.debug(msg)` / `info` / `warn` / `error(msg)` | Log at that level |
| `log.log(level, msg)` | Log at an explicit level |
| `log.enabled(level)` | Would a message at `level` be emitted? |
| `log.render(level, msg)` | The line it would print, without printing it |

### Package-level functions

`debug`, `info`, `warn`, and `error` each log through the shared default logger.
For anything other than its defaults, use a `Logger`.

### `Level`

The enum behind it all. It derives `Eq`, `Ord`, and `ToString`, so you can
compare levels directly. `level.label()` gives the padded tag (`"INFO "`) and
`level.color()` the ANSI escape.

## Requirements

Needs raven 2.18.52 or newer, where `@derive(Ord)` is what puts the levels in
order.

## License

MIT
