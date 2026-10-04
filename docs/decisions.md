# Design decisions

Running record of the design interview (started 2026-10-04). One entry per question: what
was decided and what follows from it. Hard-to-reverse calls also get an ADR in `adr/`;
vocabulary lives in `CONTEXT.md`.

## Round 1: roots

### Q1. Audience and ambition
**Decided:** The open control stack for the Topping family: community-grade, with
contributor-facing device definitions and the protocol spec as a reference.
**Follows:** Packaging, docs and safety defaults are built for strangers' hardware, not
only the maintainer's. See ADR 0004 for how this squares with "DX5 II first".

### Q2. What "way better" means
**Decided:** Five win conditions: speed (instant launch, no browser); scripting and
automation; PEQ workflow (curve editing, AutoEQ/REW import, A/B, a profile library);
Linux and macOS; openness (documented protocol, offline, no account). No specific
reliability bugs in Topping's tool were named.

### Q3. Surfaces
**Decided:** All of them are wanted: core library, `ctl` CLI, desktop GUI, tray, TUI,
browser UI. Web technology inside a native window is acceptable only if it proves good.
**Open:** the order and the v1 cut (Q10), what the browser UI is for (Q12), how "good"
is tested (Q13).

### Q4. Programmability
**Decided:** The CLI contract, resident features and embedded scripting are all wanted.
"Lightweight" describes the UI, not the absence of a background process.
**Open:** sequencing (Q10), scripting language and first use cases (Q15).

### Q5. Core language
**Decided:** Rust (ADR 0001). The maintainer does not review the code; correctness rests
on tests, the simulator and the implementing agent.
**Follows:** Verification is a first-class design concern (Q16).

### Q6. Budgets
**Decided:** Gates, to be measured on the maintainer's machine: a CLI read completes in
<= 100 ms end to end; the GUI is interactive <= 500 ms after a cold start, uses <= 150 MB
of memory, downloads at <= 30 MB and uses 0% CPU when idle. Stretch: 200 ms, 50 MB, 10 MB.
The gates may be stretched a little if a clearly better result needs it.
**Note:** These are targets. Nothing has been measured yet.

### Q7. Device scope
**Decided:** v1 supports the DX5 II only; other devices follow. Capabilities are data; no
plugin interface until a second real device exists (ADR 0004).

### Q8. Write-safety contract
**Decided:** Accepted as proposed (ADR 0002), including: guards may block a script until
it passes an explicit flag; firmware update is out for good; unknown firmware is read-only
with an explicit per-firmware opt-in.
**Open:** the numeric thresholds (what counts as a large jump) are not set yet.

### Q9. Platform verification
**Decided:** Windows and Linux can be verified on real hardware and are Tier 1. There is
no Mac, so macOS is Tier 2: built in CI, tested against the simulator, labelled
experimental until someone confirms it on hardware (ADR 0003).
