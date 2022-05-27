# `sentry-tauri`

An experimental Tauri Plugin for Sentry.

Captures and reports to Sentry:

- JavaScript errors and breadcrumbs in Tauri windows using `@sentry/browser`
- Panics and breadcrumbs in Rust via the Sentry Rust SDK
- Native crashes as minidumps

## Installation

Add the plugin as a dependency in `Cargo.toml`:

```toml
[dependencies]
sentry-tauri = {git = "https://github.com/timfish/sentry-tauri"}
```

Initialise the plugin and pass it to Tauri:

```rust
fn main() {
  sentry_tauri::init(
      sentry::release_name!(),
      |_| {
          sentry::init((
              "__YOUR_DSN__",
              sentry::ClientOptions {
                  release: sentry::release_name!(),
                  ..Default::default()
              },
          ))
      },
      |sentry_plugin| {
          tauri::Builder::default()
              .plugin(sentry_plugin)
              .run(tauri::generate_context!())
              .expect("error while running tauri application");
      },
  );
}
```

## What is going on here? 🤔

- Injects and initialises `@sentry/browser` in every web-view
- Includes a `TauriIntegration` that intercepts events and breadcrumbs and passes
  them to Rust via Tauri `invoke` API
- Tauri + `serde` + existing Sentry Rust types = Deserialisation mostly Just Works™️
- [`sentry-rust-minidump`](https://github.com/timfish/sentry-rust-minidump)
  which in turn uses the `minidumper` and `crash-handler` crates to capture
  minidumps in pure Rust and sends them via the Sentry Rust SDK

## Points to Note 📝

- This is using Sentry from git master until [this
  PR](https://github.com/getsentry/sentry-rust/pull/466) makes it into a release
- Minidumps events in Sentry are missing breadcrumbs because they are sent from
  another process. An API [like this](https://github.com/getsentry/sentry-rust/pull/469) should help fix this.

## Example App

Install dependencies:

```shell
> cd examples/basic-app
> yarn install
```

In `examples/basic-app/src-tauri/src/main.rs` replace `__YOUR_DSN__` with your DSN

Run in development mode:

```shell
> yarn tauri dev
```
