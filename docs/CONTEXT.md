# cherrytop

Open control software for Topping DACs, starting with the DX5 II: a Rust core, a `ctl` CLI,
a desktop GUI and further surfaces on top of one documented protocol.

## Language

### Structure

**Core**:
The Rust library that holds the protocol, the device model and the write-safety contract; every surface goes through it.
_Avoid_: backend, engine, SDK

**Surface**:
Anything a person or program uses to drive the core: the CLI, desktop GUI, tray, TUI, browser UI, local API, scripts.
_Avoid_: frontend, client

**Resident mode**:
cherrytop staying running in the background so that hotkeys, rules, the local API and scripts can work.
_Avoid_: daemon, service (whether it is a separate process is open: decisions Q11)

**Simulator**:
A software stand-in for a device that speaks the protocol; what tests and CI talk to instead of hardware.
_Avoid_: mock, emulator

### Devices

**Device definition**:
The data describing one device model: its USB identity, capabilities and value ranges.
_Avoid_: driver, plugin, profile

**Verified firmware**:
A firmware version on which the protocol mapping has been confirmed against real hardware.

**Support tier**:
How much evidence backs a platform. Tier 1 is verified on real hardware by a maintainer for that release; Tier 2 builds in CI and passes against the simulator, and is labelled experimental.

### Safety

**Write-safety contract**:
The rules every write must pass inside the core before it reaches a device (ADR 0002).

**Hazardous write**:
A write that can raise output level abruptly (a large upward volume jump, a PEQ profile that would clip, a level-raising setting) and therefore needs explicit confirmation.

### Programmability

**CLI contract**:
The stable, versioned `--json` output, exit codes, declarative `apply` and `watch` stream of the CLI; it works with no background process.

**Resident features**:
Hotkeys, rules and the local API; they exist only in resident mode.

**Scripting**:
User scripts run by an embedded engine in resident mode.

## Relationships

- Every **Surface** reaches a device only through the **Core**.
- The **Core** enforces the **Write-safety contract**; no **Surface** can bypass it.
- A **Device definition** lists the **Verified firmware** versions for that model; on any other version the device is read-only unless the user opts in for that exact version.
- **Resident features** and **Scripting** require **Resident mode**; the **CLI contract** does not.
- The **Simulator** is what makes a Tier 2 platform testable.

## Example dialogue

> **Dev:** "The GUI wants to push a whole PEQ profile. Does it need its own clipping check?"
> **Domain expert:** "No. That is a **hazardous write**, so the **core** checks headroom under the **write-safety contract**. The GUI only has to show the confirmation the core asks for."
> **Dev:** "And if the DAC reports a firmware version we have never seen?"
> **Domain expert:** "Then it is not **verified firmware**: every **surface** gets a read-only device until the user opts in for that exact version."

## Flagged ambiguities

- "Tier" was used for both platform support (1/2) and programmability (A/B/C). Resolved: **Support tier** keeps the word; the programmability levels are **CLI contract**, **Resident features** and **Scripting**.
- "Lightweight" was read as "no background process". Resolved: it describes the UI (fast, small, low memory); a resident process is acceptable.
- "No web app" in the README conflicts with wanting a browser UI. Open: decisions Q12.
