# TelegramProxy (Rust)

A native Rust MTProto-over-WebSocket bridge for Windows and Linux. The current source implements a local MTProto listener, obfuscated2 handshake processing, four persistent AES-CTR streams, MTProto transport packet framing, Telegram WebSocket routing, optional user-configured Cloudflare Worker / Cloudflare WebSocket routes, and direct TCP fallback.

> **Development status:** this is an early runtime, not a production-complete Telegram client proxy. It does not currently implement the Cloudflare HTTP/2 media transport, the reference project's Fake TLS listener, a desktop GUI/system tray/updater, SOCKS5/TUN, or Telegram call media. Windows 7 compatibility and real Telegram Desktop interoperability have not yet been compiled or tested. Do not assume that every Telegram feature works.

## Project layout

```text
crates/
  proxy-protocol/   MTProto obfuscated2 handshake, relay init, AES-CTR, packet splitter
  proxy-runtime/    TCP listener, WSS routes, Worker/CF route selection, TCP fallback, statistics
  telegram-proxy/  portable command-line application and JSON configuration
config/
docs/
scripts/
.github/workflows/
```

## Requirements

- Rust toolchain 1.85.0 (pinned in `rust-toolchain.toml`).
- A working Cargo registry connection for the first dependency resolution.
- For Windows x64 releases: Visual Studio C++ build tools with the MSVC linker.

The default listener is `127.0.0.1:1443`. It refuses non-loopback bind addresses to avoid exposing an unauthenticated local proxy to a network. A 16-byte MTProto secret is generated on first run and written to the application configuration file. The secret is deliberately never printed to the logs.

## Build and run

```bash
cargo fmt --all
cargo fmt --all -- --check
cargo check --workspace
cargo test --workspace
cargo clippy --workspace --all-targets --all-features -- -D warnings
cargo build --release --package telegram-proxy --bin TelegramProxy
cargo run --release --package telegram-proxy --bin TelegramProxy
```

Configuration paths:
- Windows: `%APPDATA%\\TelegramProxyRust\\config.json`
- Linux/macOS: `$HOME/.config/TelegramProxyRust/config.json`

Options: `--help`, `--version`, `--check-config`, `--config PATH`.

## Telegram Desktop setup

1. Start `TelegramProxy` and leave it running.
2. In Telegram Desktop open Settings → Advanced → Connection Type → Use custom proxy.
3. Add an **MTProto** proxy with server `127.0.0.1`, port `1443`, and the generated secret from `config.json`.
4. Enable the proxy and review the application log for the selected upstream route.

The secret is sensitive. Do not post the config file publicly.

## Route order

For each accepted client connection the current runtime tries Telegram WSS endpoints, user-configured Cloudflare Worker endpoints, user-configured Cloudflare-proxied WSS endpoints, and optionally direct TCP to the configured/default Telegram DC IP on port 443. TLS certificate verification remains enabled.

## Implemented and missing features

See `docs/IMPLEMENTATION_STATUS.md`. Successful WebSocket establishment is not proof that messages, media, long-running streams or calls have been verified end-to-end. The reference HTTP/2 media transport is not implemented. Telegram call media uses separate call/WebRTC components and is not carried by this MTProto bridge.

## Licence

See `LICENSE` and `NOTICE.md`.
