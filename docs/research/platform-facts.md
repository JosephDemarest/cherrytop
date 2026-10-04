# Platform facts: HID access, scripting engines, hotkeys, capture tools

Research snapshot, 2026-10-04. Everything here is **[reported]** (found by a research pass
with the source linked, not re-read by us) unless marked **[checked]**.

## Rust HID crates

- **hidapi** 2.6.7 (2026-08-27, MIT). Wraps the C hidapi library, built from bundled
  source. Blocking API only. No hot-plug notification, only `refresh_devices()`. On Linux
  the default hidraw backend still links libudev. The libusb backends cannot report the
  usage page. [crates.io](https://crates.io/crates/hidapi),
  [docs](https://docs.rs/hidapi/latest/hidapi/)
- **async-hid** 0.5.3 (2026-06-11, MIT). Pure Rust on Win32 or WinRT, hidraw and
  IOHIDManager; needs no libudev; has hot-plug through `HidBackend::watch()`. A macOS
  input-stall bug was fixed after the last release.
  [repo](https://github.com/sidit77/async-hid)
- **nusb** is not suitable: its docs send HID-class devices to kernel-driver libraries,
  and on Windows it needs WinUSB. [docs](https://docs.rs/nusb/latest/nusb/)

## Two programs on one HID interface

- **Windows.** hidapi opens with shared read and write access. [checked in hidapi source]
  The HID class driver keeps input queues that support more than one open file; each
  queue holds 32 reports by default and up to 512.
  [Microsoft](https://learn.microsoft.com/en-us/windows-hardware/drivers/hid/minidriver-operations)
  No Microsoft page states outright that every handle receives a copy of every report.
- **Linux.** hidraw copies each input report into every open file's queue. A full queue
  silently drops reports. There is no exclusive open.
  [hidraw.c](https://github.com/torvalds/linux/blob/master/drivers/hid/hidraw.c)
- **macOS.** C hidapi seizes the device by default [checked in hidapi source]; the Rust
  crate shares it only with the `macos-shared-device` feature.

## Permissions

- **Linux.** hidraw nodes are root-only by default. The rule to ship is
  `KERNEL=="hidraw*", ATTRS{idVendor}=="152a", ATTRS{idProduct}=="8750", TAG+="uaccess"`,
  in a file that sorts before `73-seat-late` (for example `70-cherrytop.rules`). It grants
  access to users on the local seat. Packages install rules in `/usr/lib/udev/rules.d`.
  [udev(7)](https://man7.org/linux/man-pages/man7/udev.7.html)
- **Windows.** The collections Windows opens exclusively for itself are mice, keyboards,
  pens and touch. Usage page `0x0001` with usage `0x0000` is not among them, so no admin
  rights should be needed.
  [Microsoft](https://learn.microsoft.com/en-us/windows-hardware/drivers/hid/top-level-collections-opened-by-windows-for-system-use)
- **macOS.** Whether opening this interface triggers the Input Monitoring prompt is not
  known. It needs a test on a Mac.

## WebHID

- Supported in Chrome and Edge 89+ and Opera 76+. Not in Firefox, Safari or mobile
  browsers. [caniuse](https://caniuse.com/webhid)
- Needs HTTPS and a user gesture to request a device.
- This interface is not blocked: the blocklist's Generic Desktop entries are usages
  `0x02`, `0x06`, `0x07` and `0x80` only.
  [blocklist](https://github.com/WICG/webhid/blob/main/blocklist.txt)
- On Linux, Chrome needs a udev rule to open the hidraw node.

## Embedded scripting engines

- **mlua** 0.12.2 (2026-10-03, MIT): Lua 5.1–5.5, LuaJIT and Luau, with vendored builds.
  Trap: `Lua::new()` excludes only `debug` and `ffi`, so `io`, `os` and `package` still
  load. Sandboxing needs `new_with`, or Luau's `sandbox()`, plus `set_memory_limit` and
  hooks or interrupts. [docs](https://docs.rs/mlua/latest/mlua/struct.Lua.html)
- **rhai** 1.26.1 (2026-09-10, MIT/Apache-2.0): pure Rust, sandboxed by design, guards
  against runaway scripts. [book](https://rhai.rs/book/safety/sandbox.html)
- **rquickjs** 0.14.0 (2026-09-18, MIT): QuickJS-NG, with memory and stack limits and an
  interrupt handler. [docs](https://docs.rs/rquickjs/latest/rquickjs/struct.Runtime.html)

## Global hotkeys and tray

- **global-hotkey** 0.8.0: Windows, macOS, and Linux on X11 only.
  [repo](https://github.com/tauri-apps/global-hotkey)
- **Wayland** hotkeys go through the desktop portal's GlobalShortcuts interface, where the
  user assigns the keys. KDE, GNOME 48+ and Hyprland support it; the wlroots portal does
  not. Rust client: `ashpd`.
- **tray-icon** 0.26.0: on Linux it uses AppIndicator (GTK 3) or KSNI.
  [repo](https://github.com/tauri-apps/tray-icon)

## USB capture tools

- **USBPcap.** The latest release is 1.5.4.0 from 2020; its notes list Windows 7, 8 and
  10. Installing needs admin rights and a reboot, and open issues include a blue-screen
  crash reported in 2026. Windows 11 support is not confirmed.
  [releases](https://github.com/desowin/usbpcap/releases)
- **usbmon** (Linux) logs requests between drivers and the host controller. Giving
  non-root users access exposes keyboard traffic too.
  [kernel docs](https://docs.kernel.org/usb/usbmon.html)

Capturing from the browser's developer tools (see `prior-art.md`) avoids both.

## Not verified

- That every open handle on Windows receives a copy of every input report.
- macOS behaviour: the Input Monitoring prompt, per-client report delivery, and whether
  seize is enforced.
- How much each scripting engine adds to a binary.
- USBPcap on Windows 11.
