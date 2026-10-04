# cherrytop

Fast, open control for Topping DACs: volume, PEQ and device settings over USB HID, from a
command line and a native desktop app. No account, no server, works offline.

**Status:** design stage. Nothing works yet.

## Plan

- **Protocol:** an open, documented implementation of the USB HID control protocol,
  starting with the Topping DX5 II.
- **CLI:** `cherryctl`, a single small executable for scripting and automation.
- **App:** a fast, polished desktop app for Windows and Linux, with experimental macOS
  builds.

The design decisions so far are in [docs/decisions.md](docs/decisions.md).

## Not affiliated

cherrytop is an independent project. It is not made, endorsed or supported by Topping.
"Topping" and its product names are trademarks of their owner.
