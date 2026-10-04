# GUI toolkits for a Rust desktop app

Research snapshot, 2026-10-04. Everything here is **[reported]**: found by a research pass
with the source linked, not re-read or measured by us. Nothing has been measured on our
own hardware yet; that is what the bake-off (decisions Q13, Q102) is for.

## The three bake-off candidates

### Tauri 2 (system webview with a web UI)

- Version 2.12.1 (2026-09-30), marked stable. Apache-2.0 OR MIT.
  [crates.io](https://crates.io/crates/tauri)
- Renders with WebView2 on Windows, WKWebView on macOS and WebKitGTK on Linux.
- Tray is built in; on Linux, tray click events are not emitted. Global hotkeys come from
  an official plugin built on `global-hotkey`, which is X11-only on Linux.
- Open Linux issues include Wayland protocol errors, NVIDIA window failures and Wayland
  scaling problems. [issues](https://github.com/tauri-apps/tauri/issues/10702)
- Measurements of a hello-world app:
  - Tauri's own CI: binary about 3 MB; launch to page-loaded to exit in 0.62 s on Windows
    and 0.75 s on Linux; Linux peak memory 417 MiB summed across child processes.
    [benchmark](https://github.com/tauri-apps/benchmark_results)
  - A third-party CI comparison (October 2026): about 576 ms to start on Windows and about
    307 MB for the process tree.
    [comparison](https://github.com/Elanis/web-to-desktop-framework-comparison)
- The UI is ordinary web code, so it could be reused in a browser.
- Shipped apps: GitButler, Yaak, Cap.

### Slint

- Version 1.18.1 (2026-09-21), with a stable 1.x API.
  [crates.io](https://crates.io/crates/slint)
- License: GPL-3.0-only, or the Slint royalty-free license, or commercial. The
  royalty-free license covers desktop, mobile and web apps. It requires either an
  "About Slint" entry in an About dialog or a "Made with Slint" badge on a public page,
  and it forbids embedded use. Our own source can stay MIT/Apache, but every binary
  carries Slint's terms, and anyone who forks the app must pick a Slint license
  themselves.
  [license](https://github.com/slint-ui/slint/blob/master/LICENSES/LicenseRef-Slint-Royalty-free-2.0.md),
  [FAQ](https://github.com/slint-ui/slint/blob/master/FAQ.md)
- A system tray icon is built in since 1.17, on Windows, macOS and Linux.
- Memory depends on the renderer. One real Windows app idled at about 155 MB with the
  Skia renderer on DirectX 12, about 46 MB with Skia on OpenGL, and about 20 MB with the
  software renderer. [issue](https://github.com/slint-ui/slint/issues/13470)
- Custom 2D drawing uses a `Path` element with SVG commands, and `TouchArea` reports
  pointer movement. Text input methods and screen readers worked in two independent
  surveys.
- In a browser it draws to a WebGL canvas, and upstream does not recommend it for general
  web apps. [docs](https://docs.slint.dev/latest/docs/slint/guide/platforms/web/)
- No published start-up measurements were found.
- Shipped apps: LibrePCB 2.0, and WesAudio's plugin UIs that control WesAudio hardware.

### egui / eframe

- Version 0.36.2 (2026-09-08). Upstream says the interfaces are still in flux and new
  releases will have breaking changes. MIT OR Apache-2.0.
  [repo](https://github.com/emilk/egui)
- No tray built in; it is wired up with the `tray-icon` crate.
- Accessibility goes through AccessKit; the README names Windows and macOS. Linux input
  method bugs with Fcitx5 are open.
- Custom 2D drawing is its strength: an immediate-mode painter with drag sensing.
- Measurements of a hello-world app on Windows:
  - Binary with link-time optimisation: 5.6 MB (OpenGL) or 11.3 MB (wgpu).
  - Memory: 31 MB with OpenGL, 142 MB with wgpu on DirectX 12.
  - Start-up: 100–300 ms with OpenGL; "1 to about 7 seconds" with wgpu (preliminary).
    [issue](https://github.com/emilk/egui/issues/7761)
- The same code runs in a browser, and upstream treats that as a first-class target.
- Shipped apps: the Rerun viewer, Ruffle's desktop player.

## What the numbers suggest before measuring

- **The renderer drives memory more than the toolkit does.** Two independent Windows
  reports put wgpu on DirectX 12 at about 140–175 MB idle, for both egui and Slint. On
  OpenGL or software renderers the same programs sat at about 20–55 MB. Every bake-off
  build must pin and record its renderer.
- **Tauri is likely to miss the memory gate.** The published process-tree figures
  (about 300–400 MB) are at least twice the 150 MB gate. Its start-up figures are near
  the 500 ms gate.
- **Slint and egui on OpenGL are near the stretch targets** on memory. Slint has no
  published start-up data.
- **Browser reuse** is real for Tauri and egui and not for Slint.

## Considered and left out

- **iced** 0.14.0: self-described experimental, no tray (issue open since 2019), and
  screen-reader support is still a draft. It is the alternate if a candidate drops out.
- **Dioxus** 0.7.10: its desktop mode is the same webview stack as Tauri; its native
  renderer is described upstream as experimental.
- **GPUI**: pre-1.0, the published crate lags Zed's own tree, no tray was found, and
  screen readers did not work in the surveys.
- **Qt/QML through cxx-qt** 0.10.0: text input and screen readers both worked in the 2026
  survey. Its deployed size and licensing were not researched.
- Freya, vizia, Xilem, Makepad and Floem: each is early or lacks accessibility.

## Not verified

- Slint start-up time on any OS, and its memory on Linux.
- Whether hotkeys work under Wayland in any of these stacks.
- Pointer latency at 120 Hz or more for Slint and Tauri.
- Text input and accessibility on Linux for every stack: neither survey tested Linux.
