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

## Numbers and mechanisms (decided 2026-10-04)

- **One guard for level.** The core estimates the level increase any write would cause:
  a volume change, a headphone gain switch, a preamp change, a PEQ bypass, a line-out
  mode switch, an output switch. More than 10 dB needs confirmation. A write whose effect
  cannot be estimated is treated as hazardous. The threshold can be set from 3 to 99 dB,
  in the local config file only.
- **Ramp.** A confirmed large upward change ramps at 20 dB per second. Downward changes
  are immediate.
- **Ceiling.** The volume ceiling is off by default and lives only in the local config
  file. Nothing sent over an API or from a script can raise it or the threshold.
- **Volume unit.** The raw volume unit is 0.5 dB or 1 dB depending on a device setting.
  That setting is read once per operation before any absolute volume write; if it cannot
  be read, the write is refused.
- **Whole-profile apply.** Mute, write, verify, unmute. If it fails midway the device
  stays muted and the user is told.
- **Live edits.** Streamed without muting. When a change raises the peak, preamp is
  lowered first; when it lowers the peak, the band changes first.
- **Confirmation of effect.** The device's echo must match what was sent, and hazardous
  writes are read back. Anything else is reported as "sent, not confirmed".
- **Unverified firmware.** Opt-in is per exact model and version, stored locally, after a
  compatibility read shows that every setting parses within known ranges.
- **Scenes.** Recall and save both need confirmation.

## Consequences

- Guards will sometimes block a legitimate script until it passes an explicit flag.
  Accepted.
- Firmware update is out of scope permanently, not only for v1: it is the one feature
  where a bug is unrecoverable. Users keep Topping's own tool for that.
- Each new Topping firmware release makes devices read-only in cherrytop until that
  version is verified or the user opts in.
