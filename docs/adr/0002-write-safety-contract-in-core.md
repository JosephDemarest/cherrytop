# The write-safety contract lives in the core

A bad write to a DAC can put full-scale output into headphones or speakers, or leave the
device unusable. Every write therefore passes one set of rules inside the core, so no
surface (CLI, GUI, script or API) can bypass them:

1. Every value is range-checked against the device definition. Out of range is an error,
   never a silent clamp.
2. Volume has an optional user ceiling, and a large upward jump needs an explicit flag or
   confirmation.
3. PEQ headroom is computed. A profile that would clip needs compensating preamp or an
   explicit override. A bulk apply is ordered or muted so that no intermediate state is
   louder than both endpoints.
4. A setting that can raise output level abruptly needs confirmation (`--yes` in the CLI).
5. Firmware update, factory reset and raw-opcode passthrough are not implemented.
6. A device on firmware that is not verified is read-only, unless the user opts in for
   that exact version.
7. On an unexpected response during a write: stop, re-read the device state, report. No
   blind retries.

The numeric thresholds (what counts as a large jump) are not set yet.

## Consequences

- Guards will sometimes block a legitimate script until it passes an explicit flag.
  Accepted.
- Firmware update is out of scope permanently, not only for v1: it is the one feature
  where a bug is unrecoverable. Users keep Topping's own tool for that.
- Each new Topping firmware release makes devices read-only in cherrytop until that
  version is verified or the user opts in.
