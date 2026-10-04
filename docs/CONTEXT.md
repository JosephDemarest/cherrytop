# cherrytop

Open control software for Topping DACs, starting with the DX5 II: a Rust core, the
`cherryctl` CLI, a desktop GUI and further surfaces on top of one documented protocol.

## Language

### Structure

**Core**:
The Rust library that holds the protocol, the device model and the write-safety contract; every surface goes through it.
_Avoid_: backend, engine, SDK

**Surface**:
Anything a person or program uses to drive the core: the CLI, desktop GUI, tray, browser UI, local API, scripts.
_Avoid_: frontend, client

**Transport**:
The thin adapter that moves 16-byte frames between the core and something that speaks them: a real HID device, the simulator, or a capture being replayed.
_Avoid_: driver, backend

**Operation**:
One complete exchange with a device that must not be interleaved with another, such as a read of all settings or a whole-profile apply.

**Operation lock**:
The cross-process lock a cherrytop process holds for the length of one operation, so that two processes never talk to a device at once.
_Avoid_: owner, daemon

**Resident mode**:
The GUI staying running in the tray so that hotkeys, rules, the local API and scripts can work. A per-user process, not a system service.
_Avoid_: daemon, service

**Simulator**:
A software stand-in for a device that speaks the protocol; what tests, CI and demo mode talk to instead of hardware.
_Avoid_: mock, emulator

### Devices

**Device definition**:
The data describing one device model: its USB identity, commands, capabilities and value ranges. Compiled into the binary.
_Avoid_: driver, plugin, profile

**Verified firmware**:
A firmware version on which the protocol mapping has been confirmed against real hardware.

**Support tier**:
How much evidence backs a platform. Tier 1 is verified on real hardware by a maintainer for that release; Tier 2 builds in CI and passes against the simulator, and is labelled experimental.

**Scene**:
One of the device's two stored setups, C1 and C2, recalled from the front panel, the remote or a command.

**Preset slot**:
One of the device's own stored PEQ configurations.
_Avoid_: profile

### PEQ

**Profile**:
A named PEQ setup stored as a file on the computer: filters, preamp, tags. Device-independent.
_Avoid_: preset (that is the device's word for its own slots)

**Preamp**:
The overall gain applied before the PEQ bands, lowered to make room for bands that boost.

**Headroom**:
The distance between the loudest point of a profile's summed response, preamp included, and 0 dB. Negative headroom means the profile may clip.

**Snapshot**:
A file holding every readable setting of a device at one moment, with its model and firmware.

### Safety

**Write-safety contract**:
The rules every write must pass inside the core before it reaches a device (ADR 0002).

**Level increase**:
The core's estimate of how many dB louder the output becomes because of a write. One threshold on this number guards every kind of write.

**Hazardous write**:
A write whose level increase is above the threshold or cannot be estimated, or that overwrites something stored on the device. It needs explicit confirmation.

**Disruptive write**:
A write that makes the device disconnect and reconnect, such as changing its USB audio mode.

### Evidence

**Capture**:
A recorded exchange between Topping's own software and a real device. The ground truth for the protocol (ADR 0005).
_Avoid_: trace, dump, log

**Command shape**:
The form of a command frame as seen in a capture: its type, count and index bytes, command number and checksum style. Only captured shapes are ever sent.

**Hardware run**:
A gated test run against a real device, started by the maintainer, that leaves a report in the repo.
_Avoid_: HIL, integration test

### Programmability

**CLI contract**:
The stable, versioned `--json` output, exit codes, declarative `apply` and `watch` stream of `cherryctl`; it works with no background process.

**Resident features**:
Hotkeys, rules and the local API; they exist only in resident mode.

**Scripting**:
User scripts run by an embedded engine in resident mode.

## Relationships

- Every **Surface** reaches a device only through the **Core**, over a **Transport**.
- The **Core** enforces the **Write-safety contract**; no **Surface** can bypass it.
- A process holds the **Operation lock** for exactly one **Operation**.
- A **Device definition** lists the **Verified firmware** versions for that model. On any other version the device is read-only unless the user opts in for that exact version.
- A model with no **Device definition** gets no protocol traffic at all.
- Every **Command shape** in a **Device definition** traces to a **Capture**.
- A **Profile** is checked against the **Device definition** when it is applied; a **Snapshot** belongs to one model.
- **Resident features** and **Scripting** require **Resident mode**; the **CLI contract** does not.
- A **Hardware run** on a platform is what makes that platform Tier 1 for a release.

## Example dialogue

> **Dev:** "The GUI wants to push a whole **profile**. Does it need its own clipping check?"
> **Domain expert:** "No. The **core** computes **headroom** and the **level increase** under the **write-safety contract**. If it is a **hazardous write**, the GUI only has to show the confirmation the core asks for."
> **Dev:** "And if `cherryctl` changes the volume while the GUI is open?"
> **Domain expert:** "`cherryctl` takes the **operation lock**, writes, and releases it. The GUI sees the device's echo on its own handle and updates. Nobody routes through anybody."
> **Dev:** "Can I add a command I found in someone else's notes?"
> **Domain expert:** "Not until its **command shape** shows up in a **capture** from that model."

## Flagged ambiguities

- "Tier" was used for both platform support (1/2) and programmability (A/B/C). Resolved: **Support tier** keeps the word; the programmability levels are **CLI contract**, **Resident features** and **Scripting**.
- "Lightweight" was read as "no background process". Resolved: it describes the UI (fast, small, low memory); a resident process is acceptable.
- "No web app" in the README conflicted with wanting a browser UI. Resolved: the promise is "no account, no server, works offline"; a hosted browser page is a late, optional surface.
- "Preset" and "profile" were used interchangeably. Resolved: a **Profile** is a file on the computer; a **Preset slot** is the device's own storage.
- "Owner" (a process every surface routes through) was assumed early on. Resolved: there is no owner; processes coordinate through the **Operation lock**.
