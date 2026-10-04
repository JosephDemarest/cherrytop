# Captures are ground truth; nothing unseen is ever sent

The maintainer does not review the code, and the protocol has destructive commands
(factory reset, firmware update) in the same number space as ordinary ones. On a sibling
model even a read of an unlisted register acts as a write. So the rules for what may
reach a real device are fixed here:

1. No command is sent to a real device unless that exact command shape was seen in a
   capture of Topping's own software talking to that model. No opcode scanning, no
   guessing. This is absolute, even when it blocks a feature.
2. Captures live in the repo, with serial numbers scrubbed, as ground truth. The core must
   decode every captured exchange and re-encode it byte for byte.
3. The simulator is built from the same spec and captures. Every surface is tested against
   it in CI on Windows, Linux and macOS.
4. Hardware runs happen only on the maintainer's go-ahead, reads first. The first live run
   of each new write command happens with the maintainer present, at low volume, with
   nothing valuable connected, and it restores the prior state.
5. A release is verified on real hardware on Windows and Linux before it is called
   supported.

## Consequences

- Capture sessions cost the maintainer's time. They are run from Chrome's developer tools
  on Topping's own page, so no capture driver is installed.
- A feature that Topping's software never exercises cannot be built.
- Because human review is absent, mutation testing covers the safety and codec modules,
  and a fresh-context review agent audits the write path before each release.
