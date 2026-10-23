---
title: Getting Started
description: Create, run, and understand your first GPUI Kit application.
order: -2
---

# Getting Started

You will build a desktop window with a label and a button in about 40 lines
of Rust. Along the way you will use the three pieces every GPUI Kit application
has: an application, a window, and a view.

Before you start, install Rust and the system libraries for your platform:
[Installation](./installation.md). The first build takes a few minutes because
Cargo compiles all dependencies.

## 1. Create a project

```sh
cargo new gpui-hello
cd gpui-hello
```

## 2. Add GPUI Kit

Add one dependency to `Cargo.toml`:

```toml
[dependencies]
gpui-kit = "{{gpui_kit_version}}"
```

This is the only dependency the application needs. It includes GPUI, the styled
component library, and the default icons.

## 3. Write the first view

Replace the contents of `src/main.rs` with:

```rust
use gpui_kit::component::button::{Button, ButtonVariants};
use gpui_kit::*;

struct HelloWorld;

impl Render for HelloWorld {
    fn render(&mut self, _: &mut Window, _: &mut Context<Self>) -> impl IntoElement {
        div()
            .flex()
            .flex_col()
            .size_full()
            .items_center()
            .justify_center()
            .gap_2()
            .child("Hello, World!")
            .child(
                Button::new("hello")
                    .primary()
                    .label("Click me")
                    .on_click(|_, _, _| println!("Clicked!")),
            )
    }
}

fn main() {
    application()
        .with_assets(assets::Assets)
        .run(|cx| {
            init(cx);

            open_window(WindowOptions::default(), cx, |_, cx| {
                cx.new(|_| HelloWorld)
            })
            .expect("Failed to open window");
        });
}
```

## 4. Run it

```sh
cargo run
```

A window opens with "Hello, World!" above a **Click me** button. Click the
button and `Clicked!` prints to the terminal.

## How it works

Read `main.rs` from top to bottom. A view implements `Render`. GPUI calls
`render` whenever the window redraws, and the method returns the UI as an
element tree.

```rust
struct HelloWorld;

impl Render for HelloWorld {
    fn render(&mut self, _: &mut Window, _: &mut Context<Self>) -> impl IntoElement {
        div()
            .flex()
            .flex_col()
            .size_full()
            .items_center()   // center the children horizontally
            .justify_center() // and vertically
            .gap_2()          // with space between them
            .child("Hello, World!")
            .child(
                //      ┌─── a value, built fresh on every redraw
                //      ▼
                Button::new("hello")
                    .primary()
                    .label("Click me")
                    .on_click(|_, _, _| println!("Clicked!")),
            )
    }
}
```

You style every element by chaining methods like these; [Style](./style.md)
covers the full set. Rebuilding elements every redraw is the normal pattern,
not a waste. Elements are cheap values and hold no state. A control that
needs lasting state, like the text in an input, keeps it in an
[Entity](./entity.md) owned by the view.

`main` starts the application and opens the window:

```rust
fn main() {
    application()                       // create the desktop application
        .with_assets(assets::Assets)    // and register the built-in icons
        .run(|cx| {
            init(cx);                   // prepare themes, dialogs, notifications

            open_window(WindowOptions::default(), cx, |_, cx| {
                cx.new(|_| HelloWorld)  // create the view
            })
            .expect("Failed to open window");
        });
}
```

`init` runs once, before any window opens. `open_window` wraps the view in a
[`Root`](./window.md), which hosts the overlay layers used by later guides:
dialogs, sheets, and notifications.

## A mental model

Every GPUI Kit interface has three tiers: what lives, what gets rebuilt, and
what is reusable.

<svg viewBox="0 0 720 248" role="img" aria-labelledby="mm-title" style="width: 100%; max-width: 720px; height: auto; margin: 4px 0 8px;">
  <title id="mm-title">The GPUI Kit render cycle: an Entity renders a fresh element tree built from stateless components; clicks, input and cx.notify trigger the next render</title>
  <defs>
    <marker id="mm-arrow" viewBox="0 0 10 10" refX="8" refY="5" markerWidth="7" markerHeight="7" orient="auto-start-reverse">
      <path d="M0 0L10 5L0 10z" fill="currentColor" fill-opacity="0.55" />
    </marker>
  </defs>

  <rect x="210" y="16" width="300" height="58" rx="8" fill="currentColor" fill-opacity="0.05" stroke="currentColor" stroke-opacity="0.28" />
  <text x="360" y="41" text-anchor="middle" font-size="14" font-weight="600" fill="currentColor">Entity&lt;View&gt;</text>
  <text x="360" y="59" text-anchor="middle" font-size="11.5" fill="currentColor" opacity="0.68">your view and its state live across frames</text>

  <path d="M360 74 v20" stroke="currentColor" stroke-opacity="0.55" fill="none" marker-end="url(#mm-arrow)" />
  <text x="372" y="90" font-size="11" fill="currentColor" opacity="0.68">render()</text>

  <rect x="210" y="96" width="300" height="58" rx="8" fill="currentColor" fill-opacity="0.05" stroke="currentColor" stroke-opacity="0.28" />
  <text x="360" y="121" text-anchor="middle" font-size="14" font-weight="600" fill="currentColor">Element tree</text>
  <text x="360" y="139" text-anchor="middle" font-size="11.5" fill="currentColor" opacity="0.68">fresh values every redraw, then discarded</text>

  <path d="M360 154 v20" stroke="currentColor" stroke-opacity="0.55" fill="none" marker-end="url(#mm-arrow)" />
  <text x="372" y="170" font-size="11" fill="currentColor" opacity="0.68">built from</text>

  <rect x="210" y="176" width="300" height="58" rx="8" fill="currentColor" fill-opacity="0.05" stroke="currentColor" stroke-opacity="0.28" />
  <text x="360" y="201" text-anchor="middle" font-size="14" font-weight="600" fill="currentColor">RenderOnce components</text>
  <text x="360" y="219" text-anchor="middle" font-size="11.5" fill="currentColor" opacity="0.68">Button, Input, Checkbox, stateless and reusable</text>

  <path d="M210 125 H162 Q150 125 150 113 V57 Q150 45 162 45 H208" stroke="currentColor" stroke-opacity="0.55" fill="none" marker-end="url(#mm-arrow)" />
  <text x="144" y="80" text-anchor="end" font-size="11" fill="currentColor" opacity="0.68">click, input,</text>
  <text x="144" y="95" text-anchor="end" font-size="11" fill="currentColor" opacity="0.68">cx.notify()</text>

  <rect x="540" y="184" width="170" height="42" rx="8" fill="none" stroke="currentColor" stroke-opacity="0.4" stroke-dasharray="4 3" />
  <text x="625" y="202" text-anchor="middle" font-size="11.5" font-weight="600" fill="currentColor">Entity&lt;InputState&gt;</text>
  <text x="625" y="217" text-anchor="middle" font-size="10.5" fill="currentColor" opacity="0.68">state you own and pass in</text>
  <path d="M540 205 H512" stroke="currentColor" stroke-opacity="0.45" stroke-dasharray="4 3" fill="none" marker-end="url(#mm-arrow)" />
</svg>

- **Entities live.** Your view is an `Entity` that survives between frames and
  owns state. It is the counterpart of a React component instance and its
  hooks.
- **Elements get rebuilt.** `render` returns a fresh element tree on every
  redraw, like React's virtual DOM.
- **Components are stateless.** `Button`, `Input`, and friends are `RenderOnce`
  builders. Values in, elements out, like pure function components. Lasting
  state stays with you. An `Input`'s text lives in an `Entity<InputState>`
  that you create and pass in.

A click or a `cx.notify()` marks the view dirty; GPUI calls `render` again,
and the window shows the new tree.

## Next steps

- Browse the [component catalog](../component/index.md) for buttons, inputs,
  tables, and more.
- Learn how views hold state and render it: [Entity](./entity.md),
  [Context](./context.md), [Render](./render.md).
- See a complete form with retained input state and a UI test in the
  [recipes example](https://github.com/longbridge/gpui-kit/tree/main/examples/ai_recipes).
- Target the browser instead of the desktop: [WebAssembly](./webassembly.md).

## Use GPUI Fast

By default, `gpui-kit` builds on a pinned snapshot of upstream GPUI. Enable the
`gpui-fast` feature to switch rendering, layout, and platform layers to
[GPUI Fast](https://github.com/longbridge/gpui-fast):

```toml
[dependencies]
gpui-kit = { version = "{{gpui_kit_version}}", features = ["gpui-fast"] }
```

Application code does not change. Keep the `gpui_kit::*` imports and the same
startup sequence. You don't need a `[patch]` section or a crate alias. From a
repository checkout, compare the backends with `cargo run` and
`cargo run --features gpui-fast`.

## macOS text rendering: `font-kit`

Skip this unless you depend on GPUI directly instead of `gpui-kit`. The
`gpui-kit` dependency already enables text rendering on macOS.

On a direct GPUI dependency, enable `font-kit` on `gpui-pre-platform`:

```toml
[dependencies]
gpui = { package = "gpui-pre", version = "={{gpui_pre_version}}" }
gpui_platform = { package = "gpui-pre-platform", version = "={{gpui_pre_version}}", features = ["font-kit"] }
```

Without it, the window opens but shows no text. Keep both snapshot
versions aligned.
