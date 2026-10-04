# Rust for the core and CLI

The core and the `ctl` CLI are written in Rust. We want one self-contained executable per
OS with instant start, and a type system that keeps units (dB, Hz, Q) and protocol states
apart at compile time. Python, the maintainer's usual language, was rejected because a
bundled interpreter contradicts the "lightweight, instant CLI" goal.

## Consequences

- The maintainer does not review the Rust line by line. Correctness of hardware writes
  rests on tests, the simulator and captured device traffic, so verification is designed
  in from the start rather than added later.
- GUI toolkit choices are limited to those that work with a Rust core.
